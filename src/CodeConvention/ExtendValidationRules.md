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
