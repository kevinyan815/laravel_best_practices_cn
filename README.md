# laravel_best_practices_cn

## Laravel最佳实践

### 前言
这个Repository会持续更新关于Laravel开发的最佳实践，这些实践都已经应用在了我的日常项目开发和规划中，这些分享都是随着项目开发总结出来或者是从国外开发社区学习并应用到项目开发中的实践经验。现在文章还比较少并且文章之间没什么依赖关系，你可以直接去看你想要了解的模块的最佳实践，等文章汇集的多了再来重新规划整体的目录结构和层次。 当然这更多的是我在开发时总结的一些东西，如果你有补充或者不赞同的地方欢迎提PR和Issue。 

随着时间的推移和对软件开发的理解我也很有可能会推翻之前的一些思路，对一些已分享的最佳实践进行重写。

另外如果你对Laravel内核实现原理感兴趣可以参考我写的[Laravel内核阅读指南](https://github.com/kevinyan815/Learning_Laravel_Kernel)

### 目录

- 代码规范
  - [命名约定](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/NamingConvention.md)
  - [语义话命名](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Semantics.md)
  - [单一责任原则](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/SingleResponsibility.md)
  - [使用依赖注入和Facade](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/IOCandFacade.md)
  - [应用代码层次](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/CodeLayer.md)
- 最佳实践  
  - [路由](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Route.md)
  - [控制器](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Controller.md)
  - [数据模型](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Model.md)
  - [数据库管理](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/DatabaseManage.md)
  - [Service服务类](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Service.md)
  - [事件驱动编程](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/EDP.md)
  - [测试驱动开发](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/TDD.md)
  - [扩展Validation规则](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/ExtendValidationRules.md)
  - [Rule对象应对复杂的验证规则](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/UseRuleObject.md)
  - [权限控制](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/PermissonOrPolicy.md)
  - [API请求频率限制](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/CodeConvention/Throttle.md)
- Nginx配置
  - [前后端分离项目的Nginx配置推荐](https://github.com/kevinyan815/laravel_best_practices_cn/blob/master/src/NginxConf/OneDomainHostMultiSites.conf)
 

### 其他推荐

- [Laravel内核学习指南](https://github.com/kevinyan815/Learning_Laravel_Kernel)
