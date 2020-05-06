### 应用代码分层
  
我们在写应用里的代码时根据代码负责的不同任务讲其分为五大块`Controller`, `Repository`, `Service`, `Model`, `View`。

- `Model` 数据模型， 数据模型面向的是数据层，在这里我们只关心数据表的问题，在Model中应该只定义数据与对象映射相关的属性和方法如：表名、主键名、是否让laravel管理时间字段等属性，以及模型关联、查询作用域等方法。其他与数据表无关的业务逻辑都不应出现在这里。
- `Repository` 数据逻辑访问层，由它来对接Model层，理论上有一个Model就会有一个相应的`Repository`，它的目的是将上层程序与数据层进行隔离。`Repository`是具体`interface`的实现，比如做订单相关的业务，应该有`OrderRepositoryInterface`定义`Order`数据交互流程中必须要实现的方法然后由`OrderRepository`去具体实现，之后将`OrderRepositoryInterface`和`OrderRepository`注册到服务容器中，解析时直接使用`OrderRepositoryInterface`解析出具体实现，这样上层应用程序既不需要关心数据来自哪里是`Mysql`还是`MongoDB`，同时也给项目提供了足够的灵活性。当数据来源从`Mysql`更改为`MongoDB`后，我们只需要重新写一个实现类`OrderMongoRepository`然后将服务容器里`OrderRepositoryInterface`的实现切换成`OrderMongoRepository`就好，上层完全不需要改动。
(Repository是我之前一直觉得在程序设计中特别多余而现在觉得特别重要的一个Layer，之前在Service中揉进去了Repository的职能，后续会把相关的Example也做一下修改)
- `Service` 项目的逻辑应用层，它在业务逻辑实现时会根据业务规则调用数据访问层`Repository`进行数据更新或者访问，除了数据的CRUD还会有图片上传、请求外部API获取数据、发送邮件等等其他这些功能，这些功能应该定义在`Service`层，就是所有与应用的业务相关的代码要封装在Service里，并且我们应该增强自己的领域建模能力根据业务领域抽象出具体的`Service`，以及`Service`中具有的典型方法。
- `Controller` 控制器，面向的对象是一个完整的页面或者是接口，其主要职责是作为应用与外界通信的接口，接收请求和发送响应，通过调度项目中的Service来完成请求、组合响应数据，并通过页面响应或者接口数据的形式将响应返回给客户端。
- `View` 视图， 负责渲染HTML响应，使用Laravel自带的blade模版引擎，并且应该尽量减少PHP代码。

总结：所以如果一个请求对应的业务逻辑相当复杂，我们可以通过Controller方法调用一个或者多个Service方法(单一逻辑)来完成这个复杂的逻辑，在Service方法中我们通过Repository操作来实现更新、获取数据。通过这种原则来对复杂逻辑进行解耦。

<img src="https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/images/Service%26Repository.png" width="400px" height="400px"/>


我们通过看两个例子来更好的理解代码如何分层：

#### 代码示例 

##### Example 1:

现在假设在Controller中直接用Model查询，查出所有的管理员用户:

```php
$admins = User::where('type', 'admin')->get();
```   

后来随着项目的开发你需要在不止一个Controller方法中用到这个查询， 你可以在`UserRepository`类里包装对`User`模型的访问:

```php
interface UserRepositoryInterface
{
  public function getAllAdmins()
}

class UserRepository implements UserRepositoryInterface
{
    public function getAlladmins()
    {
        return User::where('type', 'admin')->get();
    }
}
```

将UserRepositoryInterface的实现绑定到Laravel的服务容器中
```
  
<?php
namespace App\Providers;
use Illuminate\Support\ServiceProvider;
use App\Contracts\Repositories\UserRepositoryInterface;
use App\Repositories\Eloquent\UserRepository;
class RepositoryServiceProvider extends ServiceProvider
{
    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(UserRepositoryInterface::class, UserRepository::class);
    }
}
```

现在你可以在用到`UserRepository`的`Controller`中通过依赖注入`UserRepositoryInterface`, 然后通过这个UserRepository方法获取所有管理员用户:
   
```php
//Controller
public function __construct(UserRepositoryInterface $UserRepository)
{
    $this->UserRepository = $UserRepository;
}
//Controller action
public function index()
{
    $admins = $this->UserRepository->getAllAdmins();
}
```

现在我们的控制器就完全和数据层面无关了。在这里我们的数据可能来自MySQL，MongoDB或者Redis。我们的控制器不知道也不需要知道他们的区别。这样我们就可以独立于数据层来测试Web层了，将来切换存储实现也会很容易。
       
上面的例子简单说明了我们把代码分层后每个层里应该写什么类型的程序，以及代码分层后在可读性、耦合性、维护成本等方面带来的收益。接下来会有专门章节来说明各个部分的规范和常用的最佳实践。
