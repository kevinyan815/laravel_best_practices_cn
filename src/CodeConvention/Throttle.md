# API请求频率限制

在向公网提供API供外部访问数据时，为了避免被恶意攻击除了token认证最好还要给API加上请求频次限制，而在Laravel中从5.2开始框架自带的组件Throttle就支持访问频次限制了，并提供了一个Throttle中间件供我们使用，不过Throttle中间件在访问API频次达到限制后会返回一个HTML响应告诉你请求超频，在应用中我们往往更希望返回一个API响应而不是一个HTML响应，所以在文章中会提供一个自定义的中间件替换默认的Throttle中间件来实现自定义响应内容。

### 访问频次限制概述

频次限制经常用在API中，用于限制独立请求者对特定API的请求频率。例如，如果设置频次限制为每分钟1000次，如果一分钟内超过这个限制，那么服务器就会返回 `429: Too Many Attempts.`响应。

通常，一个编码良好的、实现了频率限制的应用还会回传三个响应头: `X-RateLimit-Limit`, `X-RateLimit-Remaining`和 `Retry-After`（`Retry-After`头只有在达到限制次数后才会返回）。 `X-RateLimit-Limit`告诉我们在指定时间内允许的最大请求次数， `X-RateLimit-Remaining`指的是在指定时间段内剩下的请求次数， `Retry-After`指的是距离下次重试请求需要等待的时间（s）

>注意：每个应用都会选择一个自己的频率限制时间跨度，Laravel应用访问频率限制的时间跨度是一分钟，所以频率限制限制的是一分钟内的访问次数。

### 使用Throttle中间件

让我们先来看看这个中间件的用法，首先我们定义一个路由，将中间件throttle添加到其中，throttle默认限制每分钟尝试60次，并且在一分钟内访问次数达到60次后禁止访问：

```
Route::group(['prefix'=>'api','middleware'=>'throttle'], function(){
    Route::get('users', function(){
        return \App\User::all();
    });
});
```
访问路由/api/users时你会看见响应头里有如下的信息:

>X-RateLimit-Limit: 60
X-RateLimit-Remaining: 58

如果请求超频，响应头里会返回`Retry-After`:

>Retry-After: 58
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0

上面的信息表示58秒后页面或者API的访问才能恢复正常。

#### 定义频率和重试等待时间
频率默认是60次可以通过throttle中间件的第一个参数来指定你想要的频率，重试等待时间默认是一分钟可以通过throttle中间件的第二个参数来指定你想要的分钟数。

```
Route::group(['prefix'=>'api','middleware'=>'throttle:5'],function(){
    Route::get('users',function(){
        return \App\User::all();
    });
});//频次上限5

Route::group(['prefix'=>'api','middleware'=>'throttle:5,10'],function(){
    Route::get('users',function(){
        return \App\User::all();
    });
});//频次上限5，重试等待时间10分钟
```
***注： Laravel5.5的api route文件里的路由会默认应用api中间件组，在这个中间件组中包含throttle中间件并且设定了默认每分钟60次的限制***

### 自定义Throttle中间件，返回API响应
在请求频次达到上限后Throttle除了返回那些响应头，返回的响应内容是一个HTML页面，页面上告诉我们Too Many Attempts。在调用API的时候我们显然更希望得到一个json响应，下面提供一个自定义的中间件替代默认的Throttle中间件来自定义响应信息。

首先创建一个ThrottleRequests中间件: `php artisan make:middleware ThrottleRequests`.

将下面的代码拷贝到`app/Http/Middlewares/ThrottleReuqests`文件中:
```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Cache\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

class ThrottleRequests
{
    /**
     * The rate limiter instance.
     *
     * @var \Illuminate\Cache\RateLimiter
     */
    protected $limiter;

    /**
     * Create a new request throttler.
     *
     * @param  \Illuminate\Cache\RateLimiter $limiter
     */
    public function __construct(RateLimiter $limiter)
    {
        $this->limiter = $limiter;
    }

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  \Closure $next
     * @param  int $maxAttempts
     * @param  int $decayMinutes
     * @return mixed
     */
    public function handle($request, Closure $next, $maxAttempts = 60, $decayMinutes = 1)
    {
        $key = $this->resolveRequestSignature($request);

        if ($this->limiter->tooManyAttempts($key, $maxAttempts, $decayMinutes)) {
            return $this->buildResponse($key, $maxAttempts);
        }

        $this->limiter->hit($key, $decayMinutes);

        $response = $next($request);

        return $this->addHeaders(
            $response, $maxAttempts,
            $this->calculateRemainingAttempts($key, $maxAttempts)
        );
    }

    /**
     * Resolve request signature.
     *
     * @param  \Illuminate\Http\Request $request
     * @return string
     */
    protected function resolveRequestSignature($request)
    {
        return $request->fingerprint();
    }

    /**
     * Create a 'too many attempts' response.
     *
     * @param  string $key
     * @param  int $maxAttempts
     * @return \Illuminate\Http\Response
     */
    protected function buildResponse($key, $maxAttempts)
    {
        $message = json_encode([
            'error' => [
                'message' => 'Too many attempts, please slow down the request.' //may comes from lang file
            ],
            'status_code' => 4029 //your custom code
        ]);

        $response = new Response($message, 429);

        $retryAfter = $this->limiter->availableIn($key);

        return $this->addHeaders(
            $response, $maxAttempts,
            $this->calculateRemainingAttempts($key, $maxAttempts, $retryAfter),
            $retryAfter
        );
    }

    /**
     * Add the limit header information to the given response.
     *
     * @param  \Symfony\Component\HttpFoundation\Response $response
     * @param  int $maxAttempts
     * @param  int $remainingAttempts
     * @param  int|null $retryAfter
     * @return \Illuminate\Http\Response
     */
    protected function addHeaders(Response $response, $maxAttempts, $remainingAttempts, $retryAfter = null)
    {
        $headers = [
            'X-RateLimit-Limit' => $maxAttempts,
            'X-RateLimit-Remaining' => $remainingAttempts,
        ];

        if (!is_null($retryAfter)) {
            $headers['Retry-After'] = $retryAfter;
            $headers['Content-Type'] = 'application/json';
        }

        $response->headers->add($headers);

        return $response;
    }

    /**
     * Calculate the number of remaining attempts.
     *
     * @param  string $key
     * @param  int $maxAttempts
     * @param  int|null $retryAfter
     * @return int
     */
    protected function calculateRemainingAttempts($key, $maxAttempts, $retryAfter = null)
    {
        if (!is_null($retryAfter)) {
            return 0;
        }

        return $this->limiter->retriesLeft($key, $maxAttempts);
    }
}
```
然后将`app/Http/Kernel.php`文件里的:

```
'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
```
替换成:

```
'throttle' => \App\Http\Middleware\ThrottleRequests::class,
```
就大功告成了。

### Throttle信息存储
最后再来说下，`Throttle`这些频次数据都是存储在`cache`里的，`Laravel`默认的`cache driver`是`file`也就是`throttle`信息会默认存储在框架的`cache`文件里,  如果你的`cache driver`换成redis那么这些信息就会存储在redis里，记录的信息其实很简单，`Throttle`会将请求对象的signature（以HTTP请求方法、域名、URI和客户端IP做哈希）作为缓存key记录客户端的请求次数。
