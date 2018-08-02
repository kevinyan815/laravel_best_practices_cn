在这篇文章中我们将了解到什么是“事件驱动编程”以及在Laravel中如何开始构建一个事件驱动应用，同时我们还将看到如何通过事件驱动编程来对应用程序的逻辑进行解耦。

在开始之前，先说明一下这篇文章主要是阐述事件驱动这种编程思维和理念的，所以不会涉及到Laravel Events的方方面面。如果你需要更全面地了解Laravel Events和它的各种用法可以访问[Laravel Events](https://laravel-china.org/docs/laravel/5.5/events)文档来了解详细信息。

### 何为事件驱动编程

在我们深入事件驱动应用之前，我们先看一下在维基百科里对事件驱动编程的定义:

>事件驱动编程是一种编程模式，其中的程序流由诸如用户动作（鼠标点击，按键）、传感器输出或来自其他程序/线程的消息等事件来决定确定。事件驱动编程是图形用户界面和其他应用程序（例如JavaScript Web应用程序）中使用的主要范例，用于执行某些操作来响应用户输入。

事件驱动应用程序会响应用户的动作，然后执行对应的代码来响应用户的动作。

### Laravel Events

通过上面的定义，事件是发生在应用程序中的动作。`Javascript`的事件是像鼠标点击、鼠标悬浮、按下键盘这样的用户动作。在Laravel中事件是发生在应用程序中的动作，像邮件通知、记录日志、用户注册、CRUD操作等。`Laravel Events`系统提供了简易的观察者模式实现，让开发者能够订阅和监听发生在应用中的动作。

应用中有些事件是由Laravel框架自动发起。比如说当使用`Eloquent Model`执行create、save、update或者delete操作时Laravel将分别发起`created`、`saved`、`updated`、和`deleted`事件。如果需要的话我们可以监听这些事件从而执行相应的代码来完成自己的需求。除了Laravel框架自动发起的事件，我们还可以根据自己应用的需要让Laravel发起我们自己定义的事件。比如说你可以发起一个`userRegistered`事件，在事件处理程序中发送用户验证邮件好让新注册的用户能够验证自己的邮箱。

发起一个事件并不会让应用程序执行任何相应的操作，我们必须在事件处理程序中对被发起的事件进行相应地回应。`Laravel Events`由两部分组成`Event Handler`和`Event Listener`。`Event Handler`中包含了发起事件相关的信息。`Event Listener`监听事件对象并对事件进行回应，`Event Listener`是我们实现事件逻辑的地方。在Laravel中Event类文件被存放在`app/Events`目录，Listener类文件被存放在`app/Listeners`目录。

### 为何使用事件驱动编程
我们已经了解事件驱动应用和`Laravel Events`的概念了，你可能会好奇为什么要采用事件驱动这种方法来构建你的应用程序。我们来看一下事件驱动编程带来的收益。


首先，事件是一种解耦应用程序各个方面的好方法，因为单个事件可以有多个不依赖于彼此的监听器。通过解耦，不会因为你使用了不适合域逻辑的代码而污染了代码库。其次，由于应用程序是松散耦合的，你可以轻松扩展应用程序的功能，而不必打乱/重写应用程序或应用程序的某些其他功能。

### 应用示例

现在假设新用户注册了我们的应用程序后，应用程序会给用户发送一封欢迎邮件，同时会自动给用户订阅应用上的每周新闻简报。在不应用事件驱动方式的情况下代码往往是如下这样:

    // without event-driven approach

    public function register(Request $request)
    {
        // validate input
        $this->validate($request->all(), [
          'name' => 'required',
          'email' => 'required|unique:users',
          'password' => 'required|min:6|confirmed',
        ]);

        // create user and persist in database
        $user = $this->create($request->all());

        // send welcome email
        Mail::to($user)->send(new WelcomeToSiteName($user));

        // Sign user up for weekly newsletter
        Newsletter::subscribe($user->email, [
          'FNAME': $user->fname,
          'LNAME': $user->lname
        ], 'SiteName Weekly');

        // login newly registered user
        $this->guard()->login($user);

        return redirect('/home');
    }



你可以看到发送欢迎邮件和订阅新闻简报的逻辑紧密耦合到了`register`方法里， 根据[关注点分离原则](https://baike.baidu.com/item/%E5%85%B3%E6%B3%A8%E7%82%B9%E5%88%86%E7%A6%BB)，`register`方法不应该关心发送欢迎邮件和订阅新闻简报的具体实现。你可能会觉得发送欢迎邮件和订阅新闻放到`register`方法里也没什么，但是如果在注册时除了发送邮件还要给用户发送短信呢？继续写在`register`方法里：


    public function register(Request $request)
    {
        // validate input

        // create user and persist in database

        // send welcome email
        Mail::to($user)->send(new WelcomeToSiteName($user));

        // send SMS
        Nexmo::message()->send([
          'to' => $user->phone_number,
          'from' => 'SiteName',
          'text' => 'Welcome and thanks for signup on SiteName.'
        ]);

        // Sign user up for weekly newsletter
        Newsletter::subscribe($user->email, [
          'FNAME': $user->fname,
          'LNAME': $user->lname
        ], 'SiteName Weekly');

        // login newly registered user

        return redirect('/home');
    }
    
可以看到代码库开始变得臃肿。现在让我们看看采用事件驱动编程方法如何实现上述相同的功能。

    // with event-driven approach

    public function register(Request $request)
    {
        // validate input
        $this->validate($request->all(), [
          'name' => 'required',
          'email' => 'required|unique:users',
          'password' => 'required|min:6|confirmed',
        ]);

        // create user and persist in database
        $user = $this->create($request->all());

        // fire event once user has been created
        event(new UserRegistered($user));

        // login newly registered user
        $this->guard()->login($user);

        return redirect('/home');
    }
    

一旦创建了用户，`UserRegistered`事件就会被触发。回想一下，我们之前提到，发起一个事件后应用并不会自己做任何事情，我们需要监听`UserRegistered`事件并执行必要的操作。让我们创建`UserRegistered`事件类和`SendWelcomeMail`以及`SignupForWeeklyNewsletter`监听器类：

    php artisan make:event UserRegistered
    php artisan make:listener SendWelcomeMail --event=UserRegistered
    php artisan make:listener SignupForWeeklyNewsletter --event=UserRegistered
    
事件和监听器之间的对应关系需要注册到EventServiceProvider的`$listen`属性里：

    protected $listen = [
        UserRegistered::class => [
            SendWelcomeMail::class,
            SignupForWeeklyNewsletter::class,
        ],
    ];
    
打开`app/Events/UserRegistered.php`文件更新它的构造方法：

    public $user;

    public function __construct(User $user)
    {
      $this->user = $user;
    }
    

声明$user为public，它将被传递给监听器，而监听器可以用它来执行必要的逻辑。接下来，事件监听器将在其handle方法中接收到事件实例。在handle方法中，我们可以执行响应事件的操作。

    // app/Listeners/SendWelcomeMail.php
    public function handle(UserRegistered $event)
    {
      // send welcome email
      Mail::to($event->user)->send(new WelcomeToSiteName($event->user));
    }


    // app/Listeners/SignupForWeeklyNewsletter.php
    public function handle(UserRegistered $event)
    {
      // Sign user up for weekly newsletter
      Newsletter::subscribe($event->user->email, [
        'FNAME': $event->user->fname,
        'LNAME': $event->user->lname
      ], 'SiteName Weekly');
    }

可以看到通过事件驱动的方式我们让register方法的代码尽可能的少并且专注于用户注册这件事上，其它的逻辑由`UserRegistered`事件的监听器来负责，现在如果说我们想在用户注册后发送短信给新注册的用户，我们所要做的就是创建一个新的事件监听器来监听UserRegistered事件何时被触发

    php artisan make:listener SendWelcomeSMS --event=UserRegistered
    
    // app/Listeners/SendWelcomeSMS.php
    public function handle(UserRegistered $event)
    {
      // send SMS
      Nexmo::message()->send([
        'to' => $event->user->phone_number,
        'from' => 'SiteName',
        'text' => 'Welcome and thanks for signup on SiteName.'
      ]);
    }
    
***注：记得要更新EventServiceProvider里的$listen属性***

### Conclusion


在这篇文章中，我们已经能够理解事件驱动的编程是什么，事件驱动的应用程序是什么以及Laravel事件是什么。我们还研究了事件驱动应用程序的优势。但是，像跟所有有积极影响的编程概念一样，它也有缺点。事件驱动型应用程序的主要缺点是让程序流变得复杂了，尤其一些刚接触开发的人可能很难真正理解应用程序的流程。以上面的实现为例，通过`register`方法我们并不能直观地看到程序在创建用户后会向新用户发送一封欢迎邮件，并将其注册到新闻通讯中。

所以在开发中应该根据场景创造性地使用它，利用它的优势为你的应用程序解耦，而不是过度使用它。
