
## 命名约定

遵循 [PSR 标准](http://www.php-fig.org/psr/psr-2/)。 另外，请遵循 Laravel 社区接受的命名约定：

| 类型                             | 规则                                                         | 正确示例                                | 错误示例                                            |
| -------------------------------- | ------------------------------------------------------------ | --------------------------------------- | --------------------------------------------------- |
| 控制器                   | 单数                                                         | ArticleController                       | ~~ArticlesController~~                              |
| 路由                 | 复数                                                         | articles/1                              | ~~article/1~~                                       |
| 命名路由            | 带点符号的蛇形命名                                           | users.show_active                       | ~~users.show-active, show-active-users~~            |
| 模型                        | 单数                                                         | User                                    | ~~Users~~                                           |
| hasOne 和 belongsTo 模型关联方法 | 单数                                                         | comment                                 | ~~comments, article_comment~~                |
|                                  |                                                              |                                         |                                                     |
| 其他模型关联方法 | 复数                                                         | comments                         | ~~articleComment, article_comments~~                |
| 数据表                   | 复数                                                         | article_comments                        | ~~article_comment, articleComments~~                |
| 多对多关联的中间表(Pivot table) | 按字母顺序排列的单数模型名称                                 | article_user                            | ~~user_article, articles_users~~                    |
| 数据表字段 | 小写字母、单数、蛇形命名 | meta_title                              | ~~MetaTitle; meta_titles~~        |
| 外键          | 带_id后缀的单数型名称                                      | article_id                              | ~~ArticleId, id_article, articles_id~~              |
| 主键                  | -                                                            | id                                      | ~~custom_id~~                                       |
| Migration                        | -                                                            | 2017_01_create_articles_table, 2017_01_add_name_filed_on_articles_table | ~~2017_01_01_000000_articles~~                      |
| Method Name   | 小驼峰命名                                                   | getAll                                  | ~~get_all~~                                         |
| 资源控制器中的方法 | [具体看表格](https://laravel.com/docs/master/controllers#resource-controllers) | store、index、detail… | ~~saveArticle~~                                     |
| 单元测试类中的方法 | 小驼峰命名                                                   | testGuestCannotSeeArticle               | ~~test_guest_cannot_see_article~~                   |
| 变量                     | 小驼峰命名                                                   | $articlesWithAuthor                     | ~~$articles_with_author~~                           |
| 集合(Collection)类型数据的变量名 | 具描述性的复数形式                                           | $activeUsers = User::active()->get()    | ~~$active, $data~~                                  |
| 对象                         | 具描述性的单数形式                                           | $activeUser = User::active()->first()   | ~~$users, $obj~~                                    |
| Config and language files index  | 蛇形命名                                                     | articles_enabled                        | ~~ArticlesEnabled; articles-enabled~~               |
| View文件                | 蛇形命名                                                     | show_filtered.blade.php                 | ~~showFiltered.blade.php, show-filtered.blade.php~~ |
| Config文件                 | 蛇形命名                                                     | google_calendar.php                     | ~~googleCalendar.php, google-calendar.php~~         |
| 契约 (interface) | 形容词或名词                                                 | Authenticatable                         | ~~AuthenticationInterface, IAuthentication~~        |
| Trait                            | 形容词                                                       | Notifiable                              | ~~NotificationTrait~~                               |


