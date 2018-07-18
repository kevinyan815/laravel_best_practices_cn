### 命名语义话

整个规范中我们多次强调了命名规范的事情，目的就是希望能够通过够通过合理的命名让程序能够跟语义话，
让开发人员看到代码就能大概知道功能和场景是什么。


#### 常见的错误:

- Model中属性和方法的命名一定都是针对数据层面的，代表的是数据的属性和行为，所以命名时不能出现`getXXX`类似的动宾形式词汇，
具体请看数据模型章节。


- 所有变量和方法的命名都要与当前逻辑和场景相关，比如当前变量里存放的是用户数据，那么你不能把变量名命名成不关的`$postData`，
当前逻辑里处理的是文章下面，那么调用的Service方法就应该是`setArticleOff`而不是`setPostOff`。这一条列的情况特别常见于拷贝别的项目代码到当前项目用的情况，即使拷贝过来的代码能用也请把命名更正到当前业务相关的名称。

- Controller命名也一味以`getXXX`开头，`getXXX`这个句型在开发人员中实在是太受欢迎，控制器方法的命名也是尽量以[资源路由](https://laravel-china.org/docs/laravel/5.5/controllers/1296#resource-controllers)里的风格来给方法命名。


#### 封装方法带来收益不仅仅是复用性

我们都知道封装方法可以提高代码块的复用性，其实除此之外还会提高程序的可读性。

比如项目中生成用户unique id时我们使用了如下一行代码
```angular2html
$uniqueId = uniqid();
```
但我们还是建议将其封装成一个方法, 并按照功能命名方法名称
```
function genUserIdentifier()// short for generate user identifier
{
    return uniqid();
}
```

这样我们在用到生成用户标示的地方调用这个方法就知道，此时程序生成的是一个用户标示
```angular2html
$identifier = genUserIdentifier();
```

同时根据关注点分离原则，调用他的程序不需要关注生成唯一标示的细节，即使我们需要调整生成规则时只需要修改`genUserIdentifier`里生成unique id的规则即可。
