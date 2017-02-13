# Scheduling：抽象化的Laravel任务调度

##开发背景

对于将项目用 package 打包的开发者来说，Laravel 的任务调度无法与项目一起跟随版本控制，无疑是一件非常痛苦的事情。于是我尝试继承 Laravel 中的某个类来实现目的。

Laravel 的任务调度需要写在`app/Console/Kernel.php`中，而查看命令行的入口文件`bootstrap\app.php`可以发现，它直接将`Kernel.php`这个文件给绑定了。

```php
$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
```

看来继承`Kernel.php`是行不通的。