### Model

Model只关注对数据表的抽象，模型中不能写业务代码，应该只出现与数据表属性、模型关联、查询作用域相关的代码。

#### 定义数据表的属性

任何数据表相关的属性都需要在Model中进行相应设置

```
    /**
     * Model对应的数据库连接名，不指定则使用默认连接
     *
     * @var string
     */
    protected $connection = 'lenovoweixin';

    /**
     * Model对应的table name, 在table name不遵从Laravel默认的规范时才需指定
     *
     * @var string
     */
    protected $table = 'wx_machineinfo';

    /**
     * 指定id以外的其他字段名为主键
     *
     * @var string
     */
    protected $primaryKey = 'ProductSn';

    /**
     * 指定是否需要Model自动维护时间字段 create_at 和 updated_at
     *
     * @var bool
     */
    public $timestamps = false;

    /**
     * 指定数据表的主键是否自增
     *
     * @var bool
     */
    public $incrementing = false;
    
    /**
     * 可被Model::create()方法批量赋值的字段
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password', 'email_token'
    ];
    
    /**
     * 序列化(toArray, toJson)Model对象时会隐藏的属性值
     *
     * @var array
     */
    protected $hidden = [];


```

### 模型关联

需要连表查询时不要使用`DB::join()`方法改为使用模型关联方法

#### 定义关联

```
class MachineInfo extends Model
{
    /**
     * 定义到 UserMachine的一对多模型关联
     */
    public function userMachines()
    {
        return $this->hasMany(\App\Models\UserMachine::class, 'machine_no', 'ProductSn');
    }
}
```

#### 定义反向关联

```
class UserMachine extends Model
{
    /**
     * 定义UserMachine到MachineInfo的多对一关联
     */
    public function machineInfo()
    {
        return $this->belongsTo(\App\Models\MachineInfo::class, 'machine_no', 'ProductSn');
    }
}
```

#### 使用with预加载关联

使用with预加载关联的数据，避免每次使用关联时都去查询数据库, 而且with预加载关联数据在底层是Laravel把两次查询的结果
匹配到一起的效率要高过SQL关联查询。

```
$userMachines = UserMachine::with('machineInfo')->get();
foreach ($userMachines as $machine) {
    $machine->machineInfo->productSn
}
```
整个操作只执行了两条查询, 之后Laravel会把machineInfo数据关联到匹配的userMachine数据上去
```
SELECT * FROM user_machine;

SELECT * FROM machine_info where machine_no in (1,2,3......);
```

### 查询作用域

查询作用域是针对数据表单个where查询条件的抽象，我们应该把模型常用的where条件查询封装在查询作用域中，查询作用域中的一般只包含一个where条件查询，或者有强相关性的多个where条件查询，例如

#### 定义查询作用域

```
class UserMachine extends Model
{

    /**
     * 限制查询只查询有效的设备
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeValid($query)
    {
        return $query->where('is_delete', '=', 0);
    }
    
    /**
     * 限制查询只查询在指定生产日期内的设备
     */
    public function scopeCreateTimeBetween($query, $startTime, $endTime)
    {
        return $query->where('created_at', $startTime)->where('created_at', $endTime);
    }
}
```

#### 使用查询作用域

```
//查询在指定日期内创建的有效用户设备
$userMachines = UserMachine::with('machineInfo')->valid()->createTimeBwtween($timeStart, $timeEnd)->get();
```

#### 作用域的好处

- 使查询能够更语义话，更接近英文句子，容易让人看懂。
- 解耦查询语句，让应用不再充斥着复杂的查询语句。

#### 作用域使用注意事项

- 作用域只是对一个where条件查询，或者有强相关性的多个where条件查询的封装，封装的查询条件应该尽量少而不是多。
- 查询作用域方法中不能使用get()
- 查询作用域方法的命名应该使用形容词性质的单词或者短语，不能动宾形式(getXXX), 因为查询作用域只是对查询条件的封装，具体的数据获取(get)不是它关心的。

#### 减少使用QueryBuilder(DB) ，禁止使用原生的SQL查询

与数据表的数据交互应该尽量使用数据表对应的模型而不是DB查询构建器方法，在项目中应该明令禁止使用DB直接执行SQL语句进行查询

错误的写法:

```
DB::select('SELECT * FROM users')->get()

DB::select('select * from users where id = :id', ['id' => 1]);

DB::select(SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC);
```

正确的写法:

```
User::all();

User::find(1);

//获取所有profile有效的用户发布的文章
Article::has('user.profile')->verified()->latest()->get();
```

### 用异常捕获判断Model更新是否成功用

Laravel中在如果执行SQL查询不成功会抛出`Illuminate\Database\QueryException`异常，所以我们判断数据添加、删除、更新是否成功最准确的办法是捕获这个异常，而不是去判断SQL执行后受影响的行数。
与此同时如果service方法通过catch异常的方式来回滚事务，那么在回滚后一定要再将QueryException重新抛给外部，由外部调用着再来catch。
这样做的原因是：
 - 外部调用这可以通过异常来定义响应行为
 - Service方法保持抛出异常写测试用例时才能对方法进行反向单元测试(查看以后的单元测试章节)。
 
下面是一个例子：

```
class MemberService
{
    /**
     * 恢复会员卡状态和库存
     *
     * @param \App\Models\Card $card
     * @return boolean
     */
    public function restoreCard(Card $card)
    {
        if($card->status != Card::STATUS_PREUSED) {//只有发放中状态的卡才能恢复
            return false;
        }
        DB::beginTransaction();
        try {
            $card->status = Card::STATUS_UNUSED;//将这张卡号设置为未使用
            $card->save();
            RemainingNum::firstData()->increment('remaining_num');//将剩余会员卡数加1
            DB::commit();
        } catch (QueryException $exc) {
            \Log::error('Restore Card Error: ' . $exc->getMessage());
            DB::rollback();
            throw $exc;
        }
        
        return true;
    }
}

class MemberController extends Controller
{
    public function restore(Request $request, MemberService $memService)
    {
        ......
        try {
            ......
            $memService->restoreCard($card);
        } catch (QueryException) {
            //发生错误后的响应行为
        }
        // 正确的响应行为
    }
}
```
