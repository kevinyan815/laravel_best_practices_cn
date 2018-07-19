### 使用服务容器注入依赖和Facade

创建类的对象时，不仅使得类与类之间紧密耦合，还加重了测试的复杂度。推荐改用服务容器注入依赖类对象或 facades。

坏：

```
$user = new User;
$user->create($request->all());

$redis = new \Predis\Predis(...);
$redis->get(...);
```

好：

```
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->all());

\Redis::get(...);
```
