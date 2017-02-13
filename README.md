# Scheduling 简介

在 Laravel 中，开发者要使用任务调度必须要将任务写进`app/Console/Kernel.php`文件，而无法包含在 package 中。

Scheduling 允许你在你的 package 中添加 Laravel 任务调度。

## 安装与使用

**在项目根目录下运行如下composer命令：**

```
composer require element-vip/scheduling:dev-master
```

**注册服务提供者：**

在`config/app.php`文件中 providers 数组里加入：

```
ElementVip\Scheduling\Providers\SchedulingProvider::class,
```

> 注意：请将`SchedulingProvider`在所有使用此包提供的方法定义任务调度的服务提供者之前注册！

**定义任务调度：**

在你的项目中建立任务调度文件并继承`src\Schedule\Scheduling.php`文件，并重写`schedule()`方法，具体使用方法请参考[Laravel任务调度](https://laravel-china.org/docs/5.3/scheduling)。

```php
class YourClassName extends Scheduling
{

    public function schedule()
    {
        $this->schedule->call(function () {
            // Job
        })->daily();
    }

}
```

在你项目的``中注册这个类：
```php
public function register()
{
$this->app->make('ElementVip\ScheduleList')->add(YourClassName::class);
}
```

**大功告成！**

