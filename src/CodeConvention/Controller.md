### Controller 

#### controller 的职责

Controller的职责应该是接收请求、验证请求、调用单一或者多个Service方法完成业务逻辑、返回响应。

```
class ArticleController extends Controller
{
    public function setArticleOff(Request $request, ArticleService $artService)
    {
        ...//表单验证

        $article = Article::find($request->get('article_id'));
         $this->articleService->setArticleOffline($article);

        ...//返回响应给客户端
    }
}
```

#### 把复杂的请求验证移到请求类中去

很常见但不推荐的做法：

```
class ArticleController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ]);

        ....
    }
}
```

最好是这样：

```
class ArticleController extends Controller
{
    public function store(StoreArticleRequest $request)
    {    
        ....
    }    
}


class StoreArticleRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

### 使用依赖注入

Laravel使用服务容器来解析所有的控制器。因此，你可以在控制器的构造函数中使用类型提示需要的依赖项，而声明的依赖项会自动解析并注入控制器实例中

将依赖项注入控制器能提供更好的可测试性。
```
namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
    
    public function store(Request $request)
    {
        $name = $request->name;
    }
}
```
如果控制器方法需要从路由参数中获取输入内容，只需要在其他依赖项后列出路由参数即可。比如：

```
Route::put('user/{id}', 'UserController@update');
```
```
namespace App\Http\Controllers;
use Illuminate\Http\Request;
class UserController extends Controller
{
    public function update(Request $request, User $user)
    {
        //
    }
}
```


### 使用隐式模型绑定

Laravel 会自动解析定义在路由或控制器行为中与类型提示的变量名匹配的路由段名称的 Eloquent 模型, 例如:
```
Route::Put('api/users/{user}', 'UserController@update');

class UserController extends Controller
{
    public function update(Request $request, User $user)
    {
        //
    }
}
```
***注意: 隐式模型绑定是由`\Illuminate\Routing\Middleware\SubstituteBindings`中间件来完成的,想要使用模型绑定就必须应用上这个中间件***
 
在这个例子中，由于`$user`变量被类型提示为 Eloquent 模型 User，变量名称又与 URI 中的 `{user}` 匹配，因此Laravel 会自动注入与请求 URI 中传入的 ID 匹配的用户模型实例。如果在数据库中找不到对应的模型实例，将会抛出`ModelNotFound`异常

### 依赖注入
控制器的方法支持依赖注入，控制器方法中我们常用的请求`Request`对象就是通过Laravel的服务容器注入进去的，在控制器方法中定义了路由参数、以及路由参数模型绑定后如果你还需要注入其他依赖，很简单在路由参数前添加你需要注入的依赖参数就好了。比如下面的例子
```
class UserController extends Controller
{
    public function update(Request $request, UserRepository $userRepo, User $user)
    {
        //$request request对象
        //$userRepo UserRepository对象
    }
}
```


### 保持精炼

绝不遗留「死方法」，就是没有用到的方法，控制器里的所有方法，都应该被使用到，否则应该删除；

绝不在控制器里批量注释掉代码，无用的逻辑代码就必须清除掉。
