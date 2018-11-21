# 功能测试

功能测试主要是针对项目中对外提供的各种Feature的验证，由于后端项目里子提供API所以这里讲的功能测试更多是针对API返回结果的测试。

### 生成功能测试用例
```
php artisan make:test SampleTest
```

### 发送JSON请求

因为我们的API都接受JOSN input所以在测试用例里使用`json`方法发送JSON请求。
`json`方法的参数：
- method: 定义请求方法 GET、POST、PUT、DELETE等
- uri: 接口的路由
- data: 请求体里的数据，类型为associate array如`['key' => 'name', ...]`
- headers: 请求头，类型为associate array如`['Authorization' => 'Bearer ejoYg......', ...]`

### 在测试用例里打印响应结果
写用例时经常会需要打印响应的返回值来帮助调试，使用`dump`方法可以直接输出响应，假设有如下用例：

```$xslt
class EngineerTest extends TestCase
{
    /**
     * @test
     */
    public function itShouldReceiveSuccessResponeWhenFetchEngineerInfo()
    {
        $response = $this->json('GET', '/api/wb/engineer/info');
        $response->assertStatus(200);
    }
}
```
如果用例执行失败，想要看看响应返回的是什么结果，那么只需要在发送json请求时链是调用dump()方法即可：
```
$response = $this->json('GET', '/api/wb/engineer/info')->dump();
```
然后在项目根目录下执行如下命令就能在终端里看到格式化好的响应返回值：
```
phpunit ./tests/Feature/EngineerTest.php
```

### 断言方法
列几个常用的断言方法

- assertStatus 断言HTTP响应的状态码
- assertJson 断言返回值是给定JSON的超集(包含参数中给定的JSON)
- assertJsonStructure 断言返回值的JSON符合制定的JOSN结构(支持嵌套结构的校验)

下面是使用的例子：
```
class EngineerTest extends TestCase
{
    /**
     * @test
     */
    public function itShouldReceiveSuccessResponeWhenFetchEngineerInfo()
    {
        $response = $this->json('GET', '/api/wb/engineer/info');
        $response->assertStatus(200);
        $response->assertJson(['statusCode' => 200,]);
        $response->assertJsonStructure([
            'data' => [
                'code',
                'name',
                'avatar',
                'business_id',
            ]
        ]);
    }
}
```
因为我们所有API响应的基础格式固定，接口的返回数据都放在data键的值里面，所以上面验证了返回数据data里必须包含`code`, `name`, `avatar`和`business_id`这几个字段。
更多断言响应的方法可以参考`Illuminate\Foundation\Testing\TestResponse`里的源码，代码注释里标明了每个方法的作用。

有时候接口返回的JSON数据里会返回列表数据，针对这种数据数据我们该如何断言每个列表子项目的结构呢？比如我有一个接口返回如下的JSON数据:
```
HTTP/1.1 200 Success
{
    "statusCode": 200,
    "message": {
        "info": "Success"
    },
    "data": {
        "user_avatar": "http://gavatar.com/xhfnl",
        "engineer_avatar": "http://gavatar.com/xhfnl",
        "message_list": [
            {
                "session_id": "4nksy34L-CJ",
                "chat_type": "Chat",
                "message_type": "Text",
                "media_url": "",
                "text": "1",
                "send_type": 100,
                "lenovo_id": "10087115893",
                "open_id": "ohP4Z6PuEzGB6Sr61VqWOtKZtXpE",
                "engineer_code": "A03976",
                "send_time": "2018-11-02 20:16:18",
                "emotion": 2.5,
                "client_type": "wechat",
                "access_name": "联想服务公众号"
            },
        ]
   }
}
```
那么针对`message_list`这个列表里的子元素我们应该怎么断言它们的结构呢，我们可以使用`*`来假设所有的列表内容都包含至少数组里面的内容：

```
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\ChatSession;

class ChatMessageTest extends TestCase
{
    /**
     * @test
     */
    public function itShouldFetchChatMessage()
    {
        $session = ChatSession::has('messages')->first();
        $response = $this->json('get', '/session/' . $session->id . '/messages');
        $response->assertStatus(200);
        $response->assertJson(['statusCode' => 200]);
        $response->assertJsonStructure([
            'data' => [
                'user_avatar',
                'engineer_avatar',
                'message_list' => [
                    '*' => [
                        'session_id',
                        'chat_type',
                        'message_type',
                        'media_url',
                    ]
                ],
            ]
        ]);
    }
}
```
