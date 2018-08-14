# 自定义Validator规则

项目中扩展了一些验证规则， 可以作为Laravel自带验证规则的补充应用到项目中

### 扩展自定义Validator规则的步骤

#### 创建ValidatorServiceProvider
我们先新建一个服务提供器用来存放新建的Validator规则
```
php artisan make:provider ValidatorServiceProvider
```

#### 让Laravel加载ValidatorServiceProvider
将ValidatorServiceProvider加到config/app.php配置文件的`providers`配置中
```
'providers' => [
        ......
        /**
         * Application Service Providers...
         */
        App\Providers\ValidatorServiceProvider::class,
]
```

#### 扩展Validator规则
在ValidatorServiceProvider中通过`Validator::extend()`扩展规则
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Validator;
use DB;

class ValidatorServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        //与exists规则相反, 验证的字段必须不存在于给定的数据库表中 用法: 'name' => 'not_exists:表名,字段名[字段名,字段值]'
        Validator::extend('not_exists', function($attribute, $value, $parameters) {
            $query = DB::table($parameters[0])->where($parameters[1], '=', $value);
            for ($i = 2; $i < count($parameters); $i += 2) {
                $index = $i;
                $column = $parameters[$index];
                $value = $parameters[$index + 1];
                if (strtolower($value) == 'null') {
                    $value = NULL;//解决Validator的parameters自动将NULL转为空字符串的问题
                }
                $query->where($column, '=', $value);
            }
            return $query->count() < 1;
        }, 'the filed value is duplicated');
    }

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

### 项目中扩展的Validator规则

#### not_exists:table,column

与exists规则相反, 验证的字段必须不存在于给定的数据库表中.

使用方法:
```
1. 'name' => 'not_exists:accesses,name'//验证request的name值不存在于accesses表的name字段中

2. 'name' => 'not_exists:accesses,name,deleted_at,NULL'......;//额外条件可以直接在后面附加字段和字段值
```

### 使用规则对象实现超级复杂的验证规则

```
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class ValidElementRule implements Rule
{

    /**
     * Create a new rule instance.
     * @param Illuminate\Request $request
     * @return void
     */
    public function __construct($request)
    {
        $this->request = $request;
    }


    /**
     * 判断验证规则是否通过。
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        //some complicated judgement
        if ($value == $something && $this->request->input('key') {
            // if element is not valid then return false
            return false;
        }
        
        return true
    }

    /**
     * 获取验证错误信息。
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be ......';
    }
}
```

一旦规则对象被定义好后，你可以通过将规则对象的实例传递给其他验证规则来将其附加到验证器：

```

<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use App\Rules\ValidElementRule;
class UpdateElementRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'element' => ['required', new ValidElementRule($this)]
        ];
    }
}

```
