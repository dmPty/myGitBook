# Scheduling：抽象化的Laravel任务调度

## 背景

对于将项目用 package 打包的开发者来说，Laravel 的任务调度无法与项目一起跟随版本控制，无疑是一件非常痛苦的事情。于是我尝试继承 Laravel 中的某个类来实现目的。

## 思路

Laravel 的任务调度需要写在`app/Console/Kernel.php`中，而查看命令行的入口文件`bootstrap\app.php`可以发现，它直接将`Kernel.php`这个文件给绑定了。

```php
$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
```

看来继承`Kernel.php`是行不通的。继续查看上级的类，我了解了任务调度的运行原理，所以决定参考其中的方法，重写一个`Schedule `类，并在`ServiceProvider`中来启动。

`SchedulingProvider.php`：
```php
public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->app->make('ElementVip\Scheduling\Schedule\ScheduleHandle');
        }
    }

    public function register()
    {
        $this->app->singleton('ElementVip\Scheduling\Schedule\ScheduleHandle', function ($app) {
            return new ScheduleHandle($app);
        });
    }
```

重写的`Schedule`类`src\Schedule\ScheduleHandle.php`：
```php
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Contracts\Foundation\Application;

class ScheduleHandle
{
    private $app;

    public function __construct(Application $app)
    {
        $this->app = $app;
        $this->app->booted(function () {
            $this->defineConsoleSchedule();
        });
    }

    public function defineConsoleSchedule()
    {
        $this->app->instance(
            'Illuminate\Console\Scheduling\Schedule', $scheduling = new Schedule
        );
        $this->schedule($scheduling);
    }

    public function schedule(Schedule $schedule)
    {
        $schedule->call(function () {
            // Job
        })->daily();
    }
}
```

其中的`schedule()`方法相当于`Kernel.php`中的`schedule()`，开发到这里，我终于可以在自己的 package 中添加任务调度了。

但这还没达到我当初的目的，于是我增加了一个`ScheduleList`类来储存其他 package 中注册的任务调度。

`SchedulingProvider.php`：
```php
public function register()
{
    $this->app->singleton('ElementVip\Scheduling\Schedule\ScheduleHandle', function ($app) {
        return new ScheduleHandle($app);
    });

    $this->app->singleton('ElementVip\ScheduleList', function () {
        return new ScheduleList();
    });
}
```

重写`ScheduleHandle`的`schedule()`：
```php
public function schedule(Schedule $schedule)
{
    $scheduleList = $this->app->make('ElementVip\ScheduleList');
    foreach ($scheduleList->get() as $class) {
        $instance = new $class($schedule);
        $instance->schedule();
    }
}
```

剩下需要做的就是在其他 package 的`ServiceProvider`中注册任务调度，建立文件继承`src\Schedule\Scheduling.php`，重写`schedule()`方法。

```php
public function register()
{
    $this->app->make('ElementVip\ScheduleList')->add(YourClassName::class);
}
```

这其中比较有意思的地方在于 Laravel 提供的`singleton()`方法可以绑定一个只会被解析一次的类或接口到容器中。且后面的调用都会从容器中返回相同的实例。`ScheduleList`的情况完全符合这个需求。
然后需要注意的是 Laravel 启动时先执行所有`ServiceProvider`的`register()`方法，再执行所有`ServiceProvider`的`boot()`方法，当你把`SchedulingProvider`在`app.php`中的位置放在其他`ServiceProvider`前面时，整个过程的顺序是：
> 1. 绑定`ScheduleHandle`与`ScheduleList`
2. 将其他 package 中的任务调度添加至`ScheduleList`中
3. `SchedulingProvider`执行`boot()`方法，触发`booted`事件，开始执行所有任务调度

**Thanks For Read!**