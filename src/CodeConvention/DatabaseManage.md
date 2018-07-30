## 数据库管理

### 使用迁移管理数据表
使用迁移来创建和修改数据库中的表，数据库迁移就像是数据库的版本控制，可以让团队轻松修改并共享应用程序的数据库结构。

```
#新建数据表
php  artisan make:migration migration_name --create=table_name
＃修改数据表
php  artisan make:migration migration_name --table=table_name
```
创建后在迁移类里有两个方法， up方法是执行migrate时创建或者更改数据表的操作，down是执行artisan migrate:rollback时要执行的方法， 所以down方法要对up方法里的操作做reverse。

例: 

如果使用名为`create_user_table`的迁移创建了`users`表，现在需要给`users`表增加`email`字段，这种情况下我们应该新建一个名为`add_email_on_user_table`的迁移来完成增加`email`字段的任务

```
php artisan make:migration add_email_on_user_table —table=users
```         

```
class AddEmailOnUserTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
           $table->string('email', 40);
        });
    }

    /**
     * Reverse the migrations.
     * down一定是up的逆向操作, 否则通过artisan命令无法回滚和重建数据库
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
           $table->dropColumn('email');
        });
    }
}
```
    
### 使用Seeder填充测试数据

数据库中的初始测试数据一定要通过Seeder来填充, 这样便于测试数据的管理同时在数据库重置后也能快速填充回测试数据。

此外对于主外键依赖的表的填充, 一定要控制先填充主表的测试数据, 再根据主表里的数据来填充副表的数据。
控制方法是在`DatabaseSeeder`的`run`方法中按照主外键的先后顺序调用对应的填充器

下面的例子中`QueueTableSeeder`填充数据时需要填充上面三个表的外键, 所以先填充依赖的三个表的数据:

```
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call(AccessTableSeeder::class);
        $this->call(ChannelTableSeeder::class);
        $this->call(BusinessTableSeeder::class);
        $this->call(QueueTableSeeder::class);
    }
}
```

### 常用的迁移命令:

- 通过 --force选项来更改生产数据库(用在CD部署生产环境的Job中)
   
   php artisan migrate --force
    
- 重建数据库，并用Seeder执行数据填充(用在CI测试Job中)

   php artisan migrate:fresh --seed
