---
layout: doc
title: 02-入门指南 - Codeception - 中文文档
---

# 入门指南

让我们先看一下Codeception的架构设计。 我们假定你已经[安装](http://codeception.com/install) 完了Codeception， 并且引导成功了你的第一个测试功能组件。 Codeception 生成了三个功能组件： 单元测试组件(unit)、功能测试组件(functional)、和验收测试组件(acceptance)。 我们已经在前面的章节中对它们进行了详细的描述。 现在在你的 /tests 目录中存在三个配置文件和三个目录，目录的名称与三个功能组件的名称一致(译者注：这里的名称一致，指英文名称一致)，它们都是测试中常见并且相互独立的功能。

## 表演者对象(Actors)

Codeception 中每一个主要概念都代表着测试人员的一个测试行为。 我们有单元测试人员，他们执行函数并且测试这个函数。 我们也有功能测试人员， 一个合格的功能测试人员， 将会根据已知的程序内部结构测试整个应用程序。我们还有验收测试人员，他们通过开发者提供的接口或者页面测试我们的应用程序。

Actor Classes 是通过suite配置生成的。 **他们的方法通常都来自于Codeception的模块**。每一个模块为不同的测试目标提供了预先设定的动作(action)，把它们组合起来就能够适应不同的测试环境。Codeception 的每一个模块都解决了至少90%的测试问题，所以你并不需要做重复的工作。我们认为你能预算更多的时间在写测试代码上，预算更少的时间去保证测试代码正常运行。一般情况下，验收测试人员依赖于PhpBrowser 模块，我们可以在 tests/acceptance.suite.yum文件中对PhpBrowser模块进行配置：

{% highlight yaml %}

class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser:
            url: http://localhost/myapp/
        - \Helper\Acceptance

{% endhighlight %}

在这个配置文件中你可以  开启/关闭 或者重新配置模块。
当你修改了配置后，actor 类 将会自动的更新。如果 Actor classes 没有跟你期望的那样创建或更新，你需要调用`build` 命令来重新生成：

{% highlight bash %}

$ php codecept.phar build

{% endhighlight %}


## 写一个简单的测试脚本

默认的测试用例都被写在叙述性的场景下。一个php文件要成为一个有效的测试脚本文件，那它的文件名字必须是有`Cept`后缀。

比如说，我们创建了一个文件 `tests/acceptance/SigninCept.php`

我们可以通过以下的命令去运行它们：

{% highlight bash %}

$ php codecept.phar generate:cept acceptance Signin

{% endhighlight %}

一个测试脚本将会伴随Actor class的初始化而开始。之后，在编辑器中键入  $I->  这样的的代码，然后从自动弹出列表中选择一个特有的动作函数，类似于下面的代码(这一段话其实可以理解成为把下面的代码写到php文件里面去)：

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
?>

{% endhighlight %}

现在就让我们登录到我们的网站中去。 首先，我们假定我们拥有一个通过用户名和密码就能完成授权登录的 '登录' 的页面。当我们登录成功后，我们将会在文件中看到文字：Hello，%用户名称%。现在让我们看看在Codeception中如何去写这样的测试脚本：
{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('log in as regular user');
$I->amOnPage('/login');
$I->fillField('Username','davert');
$I->fillField('Password','qwerty');
$I->click('Login');
$I->see('Hello, davert');
?>

{% endhighlight %}

在我们运行代码之前，我们要确定网站是正常运行在服务器上面的。 然后我们打开 `tests/acceptance.suite.yml` 文件 把url替换为你的网站程序的URL地址：

{% highlight yaml %}

class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser:
            url: 'http://myappurl.local'
        - \Helper\Acceptance

{% endhighlight %}

之后，我们在命令行工具中运行下面的代码：

{% highlight bash %}

$ php codecept.phar run

{% endhighlight %}

我们将会看到以下信息：

{% highlight bash %}

Acceptance Tests (1) -------------------------------
Trying log in as regular user (SigninCept.php)   Ok
----------------------------------------------------

Functional Tests (0) -------------------------------
----------------------------------------------------

Unit Tests (0) -------------------------------------
----------------------------------------------------

Time: 1 second, Memory: 21.00Mb

OK (1 test, 1 assertions)

{% endhighlight %}

获取详细的信息可以运行以下的命令：

{% highlight bash %}

$ php codecept.phar run acceptance --steps

{% endhighlight %}

我们将会看到详细的运行情况报表。

{% highlight bash %}

Acceptance Tests (1) -------------------------------
Trying to log in as regular user (SigninCept.php)
Scenario:
* I am on page "/login"
* I fill field "Username" "davert"
* I fill field "Password" "qwerty"
* I click "Login"
* I see "Hello, davert"
  OK
----------------------------------------------------  

Time: 0 seconds, Memory: 21.00Mb

OK (1 test, 1 assertions)

{% endhighlight %}

这个简单的测试能被扩展成为一个完整的测试脚本。现在我们就能模拟用户去测试你的网站。

试一下吧，少年!

## 引导器(Bootstrap)

每一个组件都有它们自己的引导器文件。 It's located in the suite directory and is named `_bootstrap.php`. It will be executed before test suite. There is also a global bootstrap file located in the `tests` directory. It can be used to include additional files.

## Cept, Cest and Test Formats

Codeception supports three test formats. Beside the previously described scenario-based Cept format, Codeception can also execute [PHPUnit test files for unit testing](http://codeception.com/docs/05-UnitTests), and Cest format.

Cest combines scenario-driven test approach with OOP design. In case you want to group a few testing scenarios into one you should consider using Cest format. In the example below we are testing CRUD actions within a single file but with a several test (one per each operation):

{% highlight php %}

<?php
class PageCrudCest
{
    function _before(AcceptanceTester $I)
    {
        // will be executed at the beginning of each test
        $I->amOnPage('/');
    }

    function createPage(AcceptanceTester $I)
    {
       // todo: write test
    }

    function viewPage(AcceptanceTester $I)
    {
       // todo: write test
    }    

    function updatePage(AcceptanceTester $I)
    {
        // todo: write test
    }
    
    function deletePage(AcceptanceTester $I)
    {
       // todo: write test
    }
}
?>

{% endhighlight %}

Such Cest file can be created by running a generator:

{% highlight bash %}

$ php codecept.phar generate:cest acceptance PageCrud

{% endhighlight %}

Learn more about [Cest format](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes) in Advanced Testing section.

## Configuration

Codeception has a global configuration in `codeception.yml` and a config for each suite. We also support `.dist` configuration files. If you have several developers in a project, put shared settings into `codeception.dist.yml` and personal settings into `codeception.yml`. The same goes for suite configs. For example, the `unit.suite.yml` will be merged with `unit.suite.dist.yml`. 

## Running Tests

Tests can be started with the `run` command.

{% highlight bash %}

$ php codecept.phar run

{% endhighlight %}

With the first argument you can run tests from one suite.

{% highlight bash %}

$ php codecept.phar run acceptance

{% endhighlight %}

To run exactly one test, add a second argument. Provide a local path to the test, from the suite directory.

{% highlight bash %}

$ php codecept.phar run acceptance SigninCept.php

{% endhighlight %}

Alternatively you can provide the full path to test file:

{% highlight bash %}

$ php codecept.phar run tests/acceptance/SigninCept.php

{% endhighlight %}

You can execute one test from a test class (for Cest or Test formats)

{% highlight bash %}

$ php codecept.phar run tests/acceptance/SignInCest.php:anonymousLogin

{% endhighlight %}

You can provide a directory path as well:

{% highlight bash %}

$ php codecept.phar run tests/acceptance/backend

{% endhighlight %}

This will execute all tests from the backend dir.

To execute a group of tests that are not stored in the same directory, you can organize them in [groups](http://codeception.com/docs/07-AdvancedUsage#Groups).

### Reports

To generate JUnit XML output, you can provide the `--xml` option, and `--html` for HTML report. 

{% highlight bash %}

$ php codecept.phar run --steps --xml --html

{% endhighlight %}

This command will run all tests for all suites, displaying the steps, and building HTML and XML reports. Reports will be stored in the `tests/_output/` directory.

To learn all available options, run the following command:

{% highlight bash %}

$ php codecept.phar help run

{% endhighlight %}

## Debugging

To receive detailed output, tests can be executed with the `--debug` option.
You may print any information inside a test using the `codecept_debug` function.

### Generators

There are plenty of useful Codeception commands:

* `generate:cept` *suite* *filename* - Generates a sample Cept scenario
* `generate:cest` *suite* *filename* - Generates a sample Cest test
* `generate:test` *suite* *filename* - Generates a sample PHPUnit Test with Codeception hooks
* `generate:phpunit` *suite* *filename* - Generates a classic PHPUnit Test
* `generate:suite` *suite* *actor* - Generates a new suite with the given Actor class name
* `generate:scenarios` *suite* - Generates text files containing scenarios from tests
* `generate:helper` *filename* - Generates a sample Helper File
* `generate:pageobject` *suite* *filename* - Generates a sample Page object
* `generate:stepobject` *suite* *filename* - Generates a sample Step object
* `generate:environment` *env* - Generates a sample Environment configuration
* `generate:groupobject` *group* - Generates a sample Group Extension


## Conclusion

We took a look into the Codeception structure. Most of the things you need were already generated by the `bootstrap` command. After you have reviewed the basic concepts and configurations, you can start writing your first scenario. 




* **Next Chapter: [AcceptanceTests >](/docs/03-AcceptanceTests)**
* **Previous Chapter: [< Introduction](/docs/01-Introduction)**