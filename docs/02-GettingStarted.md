---
layout： doc
title： 02-入门指南 - Codeception - 中文文档
---

# 入门指南

让我们先看一下Codeception的架构设计。 我们假定你已经[安装](http://codeception.com/install) 完了Codeception， 并且引导成功了你的第一个测试功能组件。 Codeception 生成了三个功能组件： 单元测试组件(unit)、功能测试组件(functional)、和验收测试组件(acceptance)。 我们已经在前面的章节中对它们进行了详细的描述。 现在在你的 /tests 目录中存在三个配置文件和三个目录，目录的名称与三个功能组件的名称一致(译者注：这里的名称一致，指英文名称一致)，它们都是测试中常见并且相互独立的功能。

## 表演者(Actors)

Codeception 中每一个主要概念都代表着测试人员的一个测试行为。 我们有单元测试人员，他们执行函数并且测试这个函数。 我们也有功能测试人员， 一个合格的功能测试人员， 将会根据已知的程序内部结构测试整个应用程序。我们还有验收测试人员，他们通过开发者提供的接口或者页面测试我们的应用程序。

Actor Classes 是通过suite配置生成的。 **他们的方法通常都来自于Codeception的模块**。每一个模块为不同的测试目标提供了预先设定的动作(action)，把它们组合起来就能够适应不同的测试环境。Codeception 的每一个模块都解决了至少90%的测试问题，所以你并不需要做重复的工作。我们认为你能预算更多的时间在写测试代码上，预算更少的时间去保证测试代码正常运行。一般情况下，验收测试人员依赖于PhpBrowser 模块，我们可以在 tests/acceptance.suite.yum文件中对PhpBrowser模块进行配置：
{% highlight yaml %}

class_name： AcceptanceTester
modules：
    enabled：
        - PhpBrowser：
            url： http：//localhost/myapp/
        - \Helper\Acceptance

{% endhighlight %}

在这个配置文件中你可以  开启/关闭 或者重新配置模块。
当你修改了配置后，actor 类 将会自动的更新。如果 Actor classes 没有跟你期望的那样创建或更新，你需要调用`build` 命令来重新生成
{% highlight bash %}

$ php codecept.phar build

{% endhighlight %}


## 写一个简单的测试场景(Scenario)例子

默认的测试用例都被写在叙述性的场景下。一个php文件要成为一个有效的场景文件，那它的文件名字必须是有`Cept`后缀。

比如说，我们创建了一个文件 tests/acceptance/SigninCept.php

我们可以通过以下的命令去运行它们：

{% highlight bash %}

$ php codecept.phar generate:cept acceptance Signin

{% endhighlight %}

一个场景一直将会伴随Actor class的初始化而开始。之后，在编辑器中键入  $I->  这样的的代码，然后从自动弹出列表中选择一个特有的动作函数，类似于下面的代码(这一段话其实可以理解成为把下面的代码写到php文件里面去)：

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
?>

{% endhighlight %}

现在就让我们登录到我们的网站中去。 首先，我们假定我们拥有一个通过用户名和密码就能完成授权登录的 '登录' 的页面。当我们登录成功后，我们将会在文件中看到文字：Hello，%用户名称%。现在让我们看看在Codeception中如何去写这样的场景测试脚本：

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

在我们运行代码之前，我们要确定网站是正常运行在服务器上面的。然后我们打开 tests/acceptance.suite.yml 文件 把url替换为你的网站程序的URL地址：

{% highlight yaml %}

class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser:
            url:'http://myappurl.local'
        - \Helper\Acceptance

{% endhighlight %}

之后，我们在命令行工具中运行下面的代码：

{% highlight bash %}

$ php codecept.phar run

{% endhighlight %}

我们将会看到以下输出：

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

这个简单的测试能被扩展成为一个完整的场景测试用例。现在我们就能仿效用户去测试你的网站。
现在我们就能模拟用户去测试你的网站。

试一下吧，少年!

## 引导器(Bootstrap)

每一个组件都有它们自己的引导器文件。它处在组件目录内，文件名为 `_bootstrap.php`。它在测试组件运行之前运行。同时也有一个公共的引导文件在 tests目录，它一般被用于加载额外的文件。

## Cept， Cest 和 测试格式(Test Formats)

Codeception支持三种测试格式。包括有刚刚详细描述过的`基于场景`的Cept格式，能测试PHPUnit的测试文件的单元测试文件，和Cest格式。

`Cest组合场景驱动`测试建议以OOP思想进行设计。万一你想要组合几种场景去进行测试你应该考虑使用Cest 格式。在下面的例子中，我们在一个文件中测试几个CRUD 动作：

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

这些Cest 文件可以通过运行下面的代码生成：

{% highlight bash %}

$ php codecept.phar generate:cest acceptance PageCrud

{% endhighlight %}

在`高级测试`部分学习更多关于[Cest 格式](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes)的知识。

## 配置

Codeception 拥有一个全局的配置文件在 codeception.yml，每一个组件也有它自己的配置文件。我们同样也支持 dist格式的 配置文件。在项目中如果我们有几个开发人员，我们可以把把 公共设置 放到 codeception.dist.yml 文件中，个人设置放入到 codeception.yml文件中。这同样适用于其它程序组件配置。比如 unit.suite.yml 将会合并 unit.suite.dist.yml 文件。

## 运行测试脚本

运行下面的命令能开始运行测试脚本。

{% highlight bash %}

$ php codecept.phar run

{% endhighlight %}

使用第一个参数能够设置运行哪一个功能组件。

{% highlight bash %}

$ php codecept.phar run acceptance

{% endhighlight %}

运行某一个测试文件，可以从功能组件目录(unit、functional、acceptance)中，提供某一个文件的名称到第二个参数。

{% highlight bash %}

$ php codecept.phar run acceptance SigninCept.php

{% endhighlight %}

或者你可以提供一个绝对的文件路径：

{% highlight bash %}

$ php codecept.phar run tests/acceptance/SigninCept.php

{% endhighlight %}

你可以只执行某一个测试类

{% highlight bash %}

$ php codecept.phar run tests/acceptance/SignInCest.php:anonymousLogin

{% endhighlight %}

你可以提供一个目录地址：

{% highlight bash %}

$ php codecept.phar run tests/acceptance/backend

{% endhighlight %}

系统会执行改目录下所有的测试。

想要执行不在同一目录中的测试组，你可以阅读[测试组](http://codeception.com/docs/07-AdvancedUsage#Groups)章节查找答案。

### 报表

如果你想要生成单元测试xml格式 报表，可以设置 --xml 选项，--html会生成 HTML报表。

{% highlight bash %}

$ php codecept.phar run --steps --xml --html

{% endhighlight %}

这条命令会执行所有的测试程序，显示每一步，并生成HTML和XML格式的报表。报表存储再 tests/_output/ 目录。

运行下面的命令，查看所有有效的选项：

{% highlight bash %}

$ php codecept.phar help run

{% endhighlight %}

## 调试

想得到更加详细的输出，可以在测试命令后面加上 --debug 选项。
在测试程序中，你可以使用 codecept_debug 函数 输出任意的信息

### 生成器

大量丰富并且有用的 Codeception 命令如下：

* `generate：cept` *suite* *filename* -  生成一个简单的 Cept 场景测试文件
* `generate：cest` *suite* *filename* - 生成一个简单的 Cest 测试文件
* `generate：test` *suite* *filename* - 生成一个简单的PHPUnit 测试脚本
* `generate：phpunit` *suite* *filename* - 生成一个PHPUnit 测试脚本类
* `generate：suite` *suite* *actor* - 通过Actor 对象的名称，生成一个新的程序组件
* `generate：scenarios` *suite* - 生成一个包含场景的文本文件
* `generate：helper` *filename* - 生成一个简单的帮助文件
* `generate：pageobject` *suite* *filename* - 生成一个简单的页面（Page）对象
* `generate：stepobject` *suite* *filename* - 生成一个简单的步骤（Step）对象
* `generate：environment` *env* - 生成一个简单环境配置文件
* `generate：groupobject` *group* - -生成一个简单的组扩展


## Conclusion

我们已经深入的了解了 Codeception的结构。通过引导器命令也生成了大部分需要的文件。回顾学到的基础概念和配置之后，你可以开始编写你的第一个场景测试脚本了。



* **下一章： [验收测试 >](/docs/03-AcceptanceTests)**
* **上一章： [< 介绍](/docs/01-Introduction)**