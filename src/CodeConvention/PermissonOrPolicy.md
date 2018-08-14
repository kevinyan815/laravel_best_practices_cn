# 用户权限控制

### 采用RBAC模型
我们使用基于角色的权限访问控制(Role-Based Access Control)来管理用户权限和角色

网上有很多开源的package让Laravel框架集成RBAC的能力，我们使用的是`spatie/laravel-permission`

- Installation

   ```
   composer require spatie/laravel-permission
   ```
   Laravel 5.5之前的版本需要手工注册服务提供器
   
   ```
    'providers' => [
        // ...
        Spatie\Permission\PermissionServiceProvider::class,
    ];

   ```
   
- Usage
 
    创建角色和权限:
 
    ```
    use Spatie\Permission\Models\Role;
    use Spatie\Permission\Models\Permission;
    
    $role = Role::create(['name' => 'writer']);
    $permission = Permission::create(['name' => 'edit articles']);
    
    ```
    将权限授权给用户:
    
    ```
    $role->givePermissionTo($permission);
    $permission->assignRole($role);
    ```
    给用户分配角色:
    
    ```
    $user->assignRole('writer');
    
    // 一次分配多个角色
    $user->assignRole('writer', 'admin');
    // 通过数据分配多个角色
    $user->assignRole(['writer', 'admin']);
    ```
    
    除了通过角色给用户分配权限外还可以将权限分配到用户身上:
    
    ```
    $user->givePermissionTo('edit articles');
    
    // 一次赋予多个权限
    $user->givePermissionTo('edit articles', 'delete articles');
    
    $user->givePermissionTo(['edit articles', 'delete articles']);
    ```
    
    获取用户的权限列表:
    ```
    // 用户直接拥有的权限
    $user->getDirectPermissions() // 也可以用模型关联的动态属性 $user->permissions;
    
    // 用户通过角色获取的权限
    $user->getPermissionsViaRoles();
    
    // 用户全部的权限
    $user->getAllPermissions();
    ```
    校验用户是否拥有给定权限和角色:
    ```
    user->hasRole('writer');
    
    $user->hasPermissionTo('edit articles');
    ```
    
更详细的使用方法请参考GitHub项目的README文件: https://github.com/spatie/laravel-permission    

### 使用Policy处理用户授权动作

项目中统一使用[授权策略](http://d.laravel-china.org/docs/5.5/authorization#policies)类来做用户授权。

所有 Policy 授权策略类 必须 继承 app/Policies/Policy.php 基类。基类类似文件如下:
```
<?php

namespace App\Policies;

use App\Models\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class BasePolicy
{
    use HandlesAuthorization;

    /**
     * Create a new policy instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    public function before($user, $ability)
    {
        //admin 拥有所有授权 ('Admin'这个角色名请根据自己系统自由更改）
        if($user->hasRole('Admin')) {
            return true;
        }
    }
}

```

#### 授权策略命名
Policy授权策略类须遵循资源路由的方式进行命名，posts 对应 /app/Policies/Post.php 。

#### 生成策略

策略是在特定模型或者资源中组织授权逻辑的类。例如，如果你的应用是一个博客，会有一个 Post 模型和一个相应的 PostPolicy 来授权用户动作，比如创建或者更新博客。

可以使用如下命令来生成策略。
```
php artisan make:policy PostPolicy
```
`make:policy`会生成空的策略类。如果希望生成的类包含基本的「CRUD」策略方法， 可以在使用命令时指定 --model 选项：
```
php artisan make:policy PostPolicy --model=Post
```
我们在项目中统一使用第二条生成包含基本策略方法的策略类

### 编写策略方法
```
<?php

namespace App\Policies;

use App\Models\User;
use App\Models\Post;

class PostPolicy extends BasePolicy
{
    /**
     * 判断指定博客能否被用户更新。
     * (因为在BasePolicy的before方法中判断用户为管理员既返回true, 
     *  所以管理员也拥有修改这篇博客的权限)
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

#### 使用授权策略
- 在Controller中使用
在控制器中使用控制器基类提供的`authorize`方法，它会自动查找与调用它的控制器方法同名的策略方法，这样控制器和授权类的方法名就统一起来了。

```
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        // 当前用户可以更新博客...
    }
```

一些动作，比如 create，并不需要指定模型实例。在这种情况下，可传递一个类名给 authorize 方法。当授权动作时，这个类名将被用来判断使用哪个策略：

```
public function create(Request $request)
{
    $this->authorize('create', Post::class);

    // 当前用户可以新建博客...
}
```

- 其他应用策略的方法

```
$user->can('update', $post)// 通过用户模型对象使用
@can('update', $post)// 在blade模版中使用
@can('create', App\Post::class)// 在blade模版中使用，不需要指定模型的动作
```

