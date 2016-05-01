---
layout: doc
title: 04-功能测试 - Codeception - 中文文档
---

# 功能测试 

直到现在，我们已经写了一些验收测试，功能测试跟验收测试区别不大，只有一个不同： 验收测试不需要一个真实的浏览器去运行测试脚本。

在一些情况下，我们在测试脚本中设置 `$_REQUEST`, `$_GET` 和 `$_POST`变量然后执行测试脚本，这些操作对于功能测试是很有价值，它能让测试脚本运行速度更快，并且在脚本出错的时候提供详细的错误信息。

Codeception支持为不同的PHP框架提供功能测试，包括以下PHP框架: Symfony2, Laravel4, Yii2, Zend Framework等等。你只需要在你的功能测试配置文件中开启相关的模块就可以进行功能测试。

这些模块为支持的PHP框架提供了一些接口，这样你的测试脚本就不需要被束缚在这些框架中。 下面是一个很简单的验收测试脚本：

{% highlight php %}

<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->click('Login');
$I->fillField('Username', 'Miles');
$I->fillField('Password', 'Davis');
$I->click('Enter');
$I->see('Hello, Miles', 'h1');
// $I->seeEmailIsSent() - special for Symfony2
?>

{% endhighlight %}

正如你看到的，你可以使用相同的测试函数去进行功能测试和验收测试。 

## 缺陷与误区

验收测试通常比功能测试要慢很多，但是功能测试不是太稳定。如果你的应用程序不是运行在一个长期存活的进程中，假如你使用了`exit`语句或者全局变量，那么就不太适合使用功能测试了。 

#### 请求头, Cookies, Sessions

对于功能测试，一个容易产生问题的地方就是测试函数使用了`headers`， `sessions`，`cookies`。大家知道，如果我们为某一个头信息调用多次`header`函数，那它就会报错。在功能测试中，我们会运行多次应用程序，这个时候，系统就会报错。

#### 共享内存

功能测试与传统方式不一样，PHP不会在应用程序处理完一个请求后结束结束自己。所有的请求处理程序都运行在一个内存容器中，它们并不是独立的。
所以 **如果你看到你的测试脚本莫名其妙的运行失败，并没有正常的执行一个单独的测试脚本.**这个时候你就要检查你的测试脚本是否是独立运行的。因为测试脚本都运行在共享内存中，很容易就被破坏运行环境。记住要保证内存干净，避免内存泄露，不要使用全局变量和静态变量。

## 开启测试框架模块

功能测试组件的脚本都放在`tests/functional`目录。
首先我们需要在组件配置文件中：`tests/functional.suite.yml`配置要开启的测试模块。在下面我们提供了支持大部分PHP框架的配置说明。

### Symfony2

To perform Symfony2 integrations you don't need to install any bundles or do any configuration changes.
You just need to include the `Symfony2` module into your test suite. If you also use Doctrine2, don't forget to include it too.
To make Doctrine2 module connect using `doctrine` service from Symfony DIC you should specify Symfony2 module as a dependency for Doctrine2.  

Example of `functional.suite.yml`

{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - Symfony2
        - Doctrine2:
            depends: Symfony2 # connect to Symfony
        - \Helper\Functional

{% endhighlight %}

By default this module will search for App Kernel in the `app` directory.

The module uses the Symfony Profiler to provide additional information and assertions.

[See the full reference](http://codeception.com/docs/modules/Symfony2)

### Laravel

[Laravel4](http://codeception.com/docs/modules/Laravel4) and [Laravel5](http://codeception.com/docs/modules/Laravel5) 
modules included, and require no configuration.


{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - Laravel5
        - \Helper\Functional

{% endhighlight %}


### Yii2

Yii2 tests are included in [Basic](https://github.com/yiisoft/yii2-app-basic) and [Advanced](https://github.com/yiisoft/yii2-app-advanced) application templates. Follow Yii2 guides to start.

### Yii

By itself Yii framework does not have an engine for functional testing.
So Codeception is the first and the only functional testing framework for Yii.
To use it with Yii include `Yii1` module into config.

{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - Yii1
        - \Helper\Functional

{% endhighlight %}

To avoid common pitfalls we discussed earlier, Codeception provides basic hooks over Yii engine.
Please set them up following [the installation steps in module reference](http://codeception.com/docs/modules/Yii1).

### Zend Framework 2

Use [ZF2](http://codeception.com/docs/modules/ZF2) module to run functional tests inside Zend Framework 2.

{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - ZF2
        - \Helper\Functional

{% endhighlight %}

### Zend Framework 1.x

The module for Zend Framework is highly inspired by ControllerTestCase class, used for functional testing with PHPUnit. 
It follows similar approaches for bootstrapping and cleaning up. To start using Zend Framework in your functional tests, include the `ZF1` module.

Example of `functional.suite.yml`

{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - ZF1
        - \Helper\Functional 

{% endhighlight %}

[See the full reference](http://codeception.com/docs/modules/ZF1)

### Phalcon 1.x

`Phalcon1` module requires creating bootstrap file which returns instance of `\Phalcon\Mvc\Application`. To start writing functional tests with Phalcon support you should enable `Phalcon1` module and provide path to this bootstrap file:

{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled:
        - Phalcon1:
            bootstrap: 'app/config/bootstrap.php'
        - \Helper\Functional

{% endhighlight %}

[See the full reference](http://codeception.com/docs/modules/Phalcon1)

## Writing Functional Tests

Functional tests are written in the same manner as [Acceptance Tests](http://codeception.com/docs/03-AcceptanceTests) with `PhpBrowser` module enabled. All framework modules and `PhpBrowser` module share the same methods and the same engine.

Therefore we can open a web page with `amOnPage` command.

{% highlight php %}

<?php
$I = new FunctionalTester;
$I->amOnPage('/login');
?>

{% endhighlight %}

We can click links to open web pages of application.

{% highlight php %}

<?php
$I->click('Logout');
// click link inside .nav element
$I->click('Logout', '.nav');
// click by CSS
$I->click('a.logout');
// click with strict locator
$I->click(['class' => 'logout']);
?>

{% endhighlight %}

We can submit forms as well:

{% highlight php %}

<?php
$I->submitForm('form#login', ['name' => 'john', 'password' => '123456']);
// alternatively
$I->fillField('#login input[name=name]', 'john');
$I->fillField('#login input[name=password]', '123456');
$I->click('Submit', '#login');
?>

{% endhighlight %}

And do assertions:

{% highlight php %}

<?php
$I->see('Welcome, john');
$I->see('Logged in successfully', '.notice');
$I->seeCurrentUrlEquals('/profile/john');
?>

{% endhighlight %}

Framework modules also contain additional methods to access framework internals. For instance, `Laravel4`, `Phalcon1`, and `Yii2` modules have `seeRecord` method which uses ActiveRecord layer to check that record exists in database.
`Laravel4` module also contains methods to do additional session checks. You may find `seeSessionHasErrors` useful when you test form validations.

Take a look at the complete reference for module you are using. Most of its methods are common for all modules but some of them are unique.

Also you can access framework globals inside a test or access Dependency Injection containers inside `Helper\Functional` class.

{% highlight php %}

<?php
namespace Helper;

class Functional extends \Codeception\Module
{
    function doSomethingWithMyService()
    {
        $service = $this->getModule('Symfony2')->grabServiceFromContainer('myservice');
        $service->doSomething();
    }
}
?>

{% endhighlight %}

Check also all available *Public Properties* of used modules to get full access to its data. 

## 错误报表

默认Codeception使用`E_ALL & ~E_STRICT & ~E_DEPRECATED` 表示错误等级。 
在功能测试中你可能想要去更改当前错误等级下依赖的错误策略。
错误报表等级能在组件配置文件中修改：
    
{% highlight yaml %}

class_name: FunctionalTester
modules:
    enabled: 
        - Yii1
        - \Helper\Functional
error_level: "E_ALL & ~E_STRICT & ~E_DEPRECATED"

{% endhighlight %}

`error_level` 能设置在 `codeception.yml` 文件中。


## 结束语

功能测试是非常好的东西，特别是你在使用了一个强大的框架的时候。通过使用功能测试你可以链接和篡改它们的内部状态。这样保证你的测试脚本很短并且很快。在其它的情况下，如果你不使用框架那不推荐你并且功能测试，如果你使用了上面没有列表的框架，那推荐你创建一个测试模块，然后分享到社区中去。




* **下一章: [单元测试 >](/docs/05-UnitTests)**
* **上一章: [< 验收测试](/docs/03-AcceptanceTests)**
