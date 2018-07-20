### 路由

#### 不要使用路由闭包

不要在路由配置文件里书写『闭包路由』或者其他业务逻辑代码，因为一旦使用将无法使用路由缓存 。

路由器要保持干净整洁，绝不 放置除路由配置以外的其他程序逻辑。

#### 使用RESTful风格的路由

|动作|	URI	|行为|	路由名称 |
|------|------|-----|------|
|GET   |	/photos |	index	|photos.index|
|GET   |	/photos/create|	create|	photos.create|
|POST|	/photos|	store|	photos.store|
|GET |	/photos/{photo} |	show|	photos.show|
|GET |	/photos/{photo}/edit|	edit|	photos.edit|
|PUT/PATCH|	/photos/{photo}	|update	|photos.update |
|DELETE |	/photos/{photo}	|destroy |	photos.destroy|

[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

下面是RESTful URI的一些例子:

- GET /zoos：列出所有动物园
- POST /zoos：新建一个动物园
- GET /zoos/ID：获取某个指定动物园的信息
- PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
- PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
- DELETE /zoos/ID：删除某个动物园
- GET /zoos/ID/animals：列出某个指定动物园的所有动物
- DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

创建资源路由

```
php artisan make:controller PhotoController --resource
```

给资源控制器注册一个资源路由
```
Route::resource('photos', 'PhotoController');
```

声明用于 APIs 的资源路由 (排除显示 HTML 模板的路由（如 create 和 edit ）)
```
Route::apiResource('photo', 'PhotoController');
```

声明资源路由时，你可以指定控制器处理的部分行为，而不是所有默认的行为：

```
Route::resource('photo', 'PhotoController', ['only' => [
    'index', 'show'
]]);


Route::resource('photo', 'PhotoController', ['except' => [
    'create', 'store', 'update', 'destroy'
]]);

```

#### 路由缓存

>{note} 基于闭包的路由不能被缓存。如果要使用路由缓存，你必须将所有的闭包路由转换成控制器类路由。

如果你的应用只使用了基于控制器的路由，那么你应该充分利用 Laravel 的路由缓存。使用路由缓存将极大地减少注册所有应用路由所需的时间。某些情况下，路由注册的速度甚至可以快一百倍。要生成路由缓存，只需执行 Artisan 命令 route:cache：
```
php artisan route:cache
```
运行这个命令之后，每一次请求的时候都将会加载缓存的路由文件。如果你添加了新的路由，你需要生成一个新的路由缓存。因此，你应该只在生产环境运行`route:cache`命令：

你可以使用`route:clear`命令清除路由缓存：
```
php artisan route:clear
```
