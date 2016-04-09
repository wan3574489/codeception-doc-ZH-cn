---
layout: doc
title: 01-介绍 - Codeception - 中文文档
---

# 介绍

对Web应用程序进行测试这个想法一直存在。 你没有办法睡一个安稳觉，特别是在你不确定你最后一次提交的代码是否会让整个Web应用程序当机的时候。
如果你的Web应用程序覆盖了测试代码，那会给你在程序运行的稳健性上充足的信心。

在大部分情况下，测试后谁都无法保证Web应用程序100%的正常工作。针对复杂的应用程序，你无法预测所有正常或者异常情况发生，但是使用测试，你能保证大部分重要的功能都能正常的工作。

我们提供了很多方式去测试你的应用程序。比较流行的例子是 [单元测试](http://en.wikipedia.org/wiki/Unit_testing)。 对于Web应用程序，使用单元测试单独的测试控制器或数据模型并不能保证你的应用程序能正常工作。当你想测试你整个应用程序的时候，你应该用功能测试或验收测试。

Codecoeption 测试框架分了三种测试等级。你可以使用框架提供的工具去编写单元测试、功能测试和验收测试。

让我们看看一些测试模型。

### 验收测试

如何让你的客户、管理者、测试人员、或者其它非技术人员知道你的网站正常工作？ 打开浏览器，链接你的网站，点击链接，输入表单提交然后看看输入的内容是否显示在网站上。这些人不知道你使用的框架、数据库、web服务器和程序语言，也不知道为什么Web应用程序没有按预期那样工作。

验收测试能够从用户的角度去帮你测试一些标准但是不复杂的功能。运行所有场景的验收测试，并且没有任何错误，能让你对Web应用程序更加有信心。

请注意,  **所有的网站系统** 都能被验收测试所覆盖。不管你使用的是CMS还是框架。

#### 简单的验收测试脚本

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->amOnPage('/');
$I->click('Sign Up');
$I->submitForm('#signup', ['username' => 'MilesDavis', 'email' => 'miles@davis.com']);
$I->see('Thank you for Signing Up!');
?>

{% endhighlight %}

#### 优点

* 可以测试任意的web网站
* 能测试javascript和Ajax请求
* 能把结果显示给你的客户或项目管理人员
* 最稳定：代码或技术变化对测试脚本的影响较小

#### 确定
* 运行速度较慢:需要运行浏览器和对数据库重建
* 检查较少会导致错误的结果
* 非常非常慢
* 运行不稳定:页面渲染和javascript错误会导致无法预知的结果


### 功能测试

功能测试能检测没有运行在服务器上的的应用程序，能让我们看到详细的异常信息，运行速度更快，并且能检测数据库的值是否跟我们期望的一样。

功能测试将模拟一个web请求($_GET and $_POST 变量) ，然后发送给应用程序，最终应用程序响应并返回一段HTML代码。在功能测试中你能为响应下断言，并且你能检测你的数据库是否正确的插入到数据库中。

为了能进行功能测试，你的应用程序要准备好运行在一个测试环境中。 Codeception 为热门的PHP框架提供了基础支持，或者你也可以自己写一个。

#### 简单的功能测试

{% highlight php %}

<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->click('Sign Up');
$I->submitForm('#signup', ['username' => 'MilesDavis', 'email' => 'miles@davis.com']);
$I->see('Thank you for Signing Up!');
$I->seeEmailSent('miles@davis.com', 'Thank you for registration');
$I->seeInDatabase('users', ['email' => 'miles@davis.com']);
?>

{% endhighlight %}

#### 优点

* 跟验收测试很像，但是更快
* 能提供更多详细的报表
* 依然能把结果显示给你的客户或项目管理人员
* 更加稳定:只有骨干代码的改变或者移动到其他框架能让测试代码无法运行。

#### 确定

* 无法测试javascript和Ajax
* 因为使用了模拟浏览器，会导致你得到较多错误的结果
* 必须要一个框架

### 单元测试

在代码片段组合之前进行测试是非常重要的。单元测试这种方式能确定一些隐藏很深的特性能正常工作，即使该特性并没有进行过功能测试和验收测试。通过单元测试也能证明产品的可靠性和代码的健壮性。

Codeception 是基于 [PHPUnit](http://www.phpunit.de/)创建的. 如果你曾经写过PHPUnit的单元测试，那你现在依然可以在Codeception中按照PHPUnit的方式去写。Codeception可以无错的执行标准的PHPUnit的单元测试，但是在这之前你需要使用Codeception提供的一些不错的构建工具让你写的单元测试更简单和干净。

没有任何经验的开发者也应该知道什么是测试，并且知道如何进行测试。 需求和代码很容易就会改变，这个时候相对应的单元测试代码也应该马上更新。。为了更好的让你理解测试场景，第一步你需要更新下面的代码到一个新的测试行为脚本中。

#### 简单的集成测试

{% highlight php %}

<?php
function testSavingUser()
{
    $user = new User();
    $user->setName('Miles');
    $user->setSurname('Davis');
    $user->save();
    $this->assertEquals('Miles Davis', $user->getFullName());
    $this->unitTester->seeInDatabase('users', ['name' => 'Miles', 'surname' => 'Davis']);
}
?>

{% endhighlight %}

#### 优点

* 速度最快(当然，在当前的例子中，你依然需要进行数据库重建)
* 能覆盖所有的功能特性
* 能测试稳定的应用程序核心代码
* 如果你写单元测试你会被认为是一个非常优秀的开发者 :)

#### 缺点

* 无法测试连接
* 不稳定:对代码的修改非常敏感

## 结束语

尽管TDD非常流行，但是也并不是所有的PHP开发者会为他们的程序编写自动化测试脚本。Codeception框架让你感觉开发测试脚本是很有趣的一件事。它支持单元测试、功能测试、集成测试和验收测试。

Codeception是一个BDD测试框架。所有的Codeception测试脚本全部采用叙述性规则进行编写。通过查看测试脚本的反馈内容，你能很清晰的知道测试何时开始、测试脚本如何执行。拥有很多断言的复杂测试脚本可以写在一个简单的基于DSL(领域特定语言)的PHP 文件中。




* **下一章: [入门 >](/docs/02-GettingStarted)**
