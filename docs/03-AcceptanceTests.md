---
layout: doc
title: 03-验收测试 - Codeception - 中文文档
---

# 验收测试

验收测试能被非技术人员执行。该人员可能是你的测试小组成员、管理者或是任意人员。如果正在开发一个web应用程序(很大可能性就是)，测试人员只需要使用浏览器就能检测你的站点是否正确运行。每一次网站变动后，你可以重新运行验收测试脚本。Codeception保持测试的干净和简单，犹如记录验收测试人员输入文字一样。

通过Codeception对CMS系统或框架 进行测试并没有什么不同。你也可以方便的对由其它语言构建的站点进行测试，例如Java、.NET或者其它。为你的网站进行验收测试永远都是一个好主意。每次修改之后，至少要确定所修改的功能是能正常运行的。

## 简单测试脚本

很大的可能，你想要进行的第一个测试就是登录。写一个这样的测试脚本，我们需要有PHP和HTML相关的知识。

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('sign in');
$I->amOnPage('/login');
$I->fillField('username', 'davert');
$I->fillField('password', 'qwerty');
$I->click('LOGIN');
$I->see('Welcome, Davert!');
?>

{% endhighlight %}

这个测试脚本的结果很大的可能性会展现给非技术人员阅读。Codeception能很'平滑'的移植这个测试脚本，把运行情况和结果转换为英文显示出来 ：

{% highlight bash %}

I WANT TO SIGN IN
I am on page '/login'
I fill field 'username', 'davert'
I fill field 'password', 'qwerty'
I click 'LOGIN'
I see 'Welcome, Davert!'

{% endhighlight %}

可以通过下面的命令行来进行转化：

{% highlight bash %}

$ php codecept.phar generate:scenarios

{% endhighlight %}

转化的结果以文本格式存储在 ___output__ 目录中。

**这个测试脚本会被PHP Browser或者加载了Selenium WebDriver的浏览器执行**。 我们开始编写第一个运行在PhpBrowser的验收测试脚本。

## PHP Browser

这是一种快速运行验收测试脚本的方法，因为它并不需要运行一个真实的浏览器。 我们使用了一个PHP web解析器来模拟一个浏览器它发送请求，获取并且解析响应。Codeception 使用 [Guzzle](http://guzzlephp.org) 和 Symfony BrowserKit 对 网页进行处理。 请注意，你无法测试真实可见的节点和javascript交互脚本。PHPBrowser 好的地方是你能够在加载了PHP和CURL的任意的环境中运行。

PHPBrowser常见的弊端：

* 你可以点击所有有效的链接和表单提交按钮
* 你无法填充表单
* 你无法测试JavaScript交互脚本：模态窗口，时间控件和其它的东西。

在开始之前，我们需要确保网站已经正确的运行在本地服务器上。然后在验收测试组件配置文件中指定URL参数 (`tests/acceptance.suite.yml`)。

{% highlight yaml %}

class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser:
            url: {{your site url}}
        - \Helper\Acceptance

{% endhighlight %}

首先， 我们在 __tests/acceptance__ 目录中创建一个 'Cept' 格式的文件。文件名  __SigninCept.php__。我们把下面的代码写入到文件中。

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('sign in with valid account');
?>

{% endhighlight %}

`wantTo`方法描述你的测试脚本所处的位置。它们是额外附加的公共方法，为了确保Codeception测试脚本是行为驱动开发(BDD Story)。 如果你曾经在Cherkin中编写行为驱动测试脚本，那你就能写下面的BDD Story(译者注：如果你要需要测试取钱的功能，那么故事就是去取钱，前期需要写出故事背景，例如你是谁，你在哪里，你想取多少钱等，然后就是是否取钱成功)：

{% highlight bash %}

As an Account Holder
I want to withdraw cash from an ATM
So that I can get money when the bank is closed

{% endhighlight %}

在Codeception这么些:

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->am('Account Holder'); 
$I->wantTo('withdraw cash from an ATM');
$I->lookForwardTo('get money when the bank is closed');
?>

{% endhighlight %}

在描述完故事背景之后，开始编写一个测试脚本。

 `$I` 对象在所有的交互中都是非常有用的。这个$I 对象中的方法来自于 PhpBrowser 模块。我们会在下面简短的描述一下:

{% highlight php %}

<?php
$I->amOnPage('/login');
?>

{% endhighlight %}

我们假定所有的 `am` 方法应该描述正要开始的环境。 例子中的`amOnPage`方法设置当前了正在测试 __/login__ 页面。

通过 `PhpBrowser` 你能点击链接并且填充表单。很有可能大部分情况下你都会调用那些操作。

#### Click

模拟点击一个有效的锚点。锚点中href指向的页面会被打开。你可以通过链接的名称、一个有效的css选择器或者Xpath找到需要点击的锚点。

{% highlight php %}

<?php
$I->click('Log in'); 
// CSS selector applied
$I->click('#login a');
// XPath
$I->click('//a[@id=login]');
// Using context as second argument
$I->click('Login', '.nav');
?>

{% endhighlight %}

Codeception尝试着通过文本、名称、css或Xpath去定位节点。你可以传递数组参数来单位节点。我们称之为 **严格定位**。 可以通过下面的参数来进行严格定位：

* id
* name
* css
* xpath
* link
* class

{% highlight php %}

<?php
// By specifying locator type
$I->click(['link' => 'Login']);
$I->click(['class' => 'btn']);
?>

{% endhighlight %}

在点击链接之前，你可以检测这个链接是否真实存在页面中。调用`seeLink`方法来执行检测的操作 。.

{% highlight php %}

<?php
// checking that link actually exists
$I->seeLink('Login');
$I->seeLink('Login','/login');
$I->seeLink('#login a','/login');
?>

{% endhighlight %}


#### 表单

在测试网站系统的时候，点击链接不会占用很多时间。如果你的网站只由链接组成，那你根本不需要进行自动化测试。大部分的工作都在测试表单。

Codeception提供几种方式对表单进行测试。

让我们在Codeception中提交一个简单的表单。

{% highlight html %}

<form method="post" action="/update" id="update_form">
     <label for="user_name">Name</label>
     <input type="text" name="user[name]" id="user_name" />
     <label for="user_email">Email</label>
     <input type="text" name="user[email]" id="user_email" />     
     <label for="user_gender">Gender</label>
     <select id="user_gender" name="user[gender]">
          <option value="m">Male</option>
          <option value="f">Female</option>
     </select>     
     <input type="submit" name="submitButton" value="Update" />
</form>

{% endhighlight %}

从用户的角度来看，一个表单由多个域组成，它们会被被填充，然后点击更新按钮提交。

{% highlight php %}

<?php
// we are using label to match user_name field
$I->fillField('Name', 'Miles');
// we can use input name or id
$I->fillField('user[email]','miles@davis.com');
$I->selectOption('Gender','Male');
$I->click('Update');
?>

{% endhighlight %}

通过labels匹配域，你应该为label标签写一个`for`属性。

从开发者的角度来看，提交表单只是发送一个有效的POST请求到服务器。有些时候填充表单，然后点击‘提交’按钮是一件很简单的事情。
可以把测试脚本改写成为下面的样子：

{% highlight php %}

<?php
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm'
)));
?>

{% endhighlight %}

`submitForm` 方法并没有仿真为一个真实的用户行为，但是该很适合不需要格式化处理的表单，例如labels没有设置、输入域没有一个干净的名称、编写不标准的id或者表单是由javascript提交的。

默认情况下，submitForm 方法不会为按钮提交数据。最后一个参数允许指定按钮的值应该被提交，或者按钮的值会被暗中加入到第二个参数中。

{% highlight php %}

<?php
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm'
)), 'submitButton');
// this would be the same effect, but the value has to be implicitly specified
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm',
	 'submitButton' => 'Update'
)));
?>

{% endhighlight %}

#### 断言

在PHP 浏览器中你能够测试网页的内容。大部分情况下，你仅仅只需要检查在页面中是否包含某些文字或者节点是否存在。

最有用的方法就是 `see`.

{% highlight php %}

<?php
// We check that 'Thank you, Miles' is on page.
$I->see('Thank you, Miles');
// We check that 'Thank you Miles' is inside 
// the element with 'notice' class.
$I->see('Thank you, Miles', '.notice');
// Or using XPath locators
$I->see('Thank you, Miles', "descendant-or-self::*[contains(concat(' ', normalize-space(@class), ' '), ' notice ')]");
// We check this message is not on page.
$I->dontSee('Form is filled incorrectly');
?>

{% endhighlight %}

你可以检查在页面中节点是否存在。

{% highlight php %}

<?php
$I->seeElement('.notice');
$I->dontSeeElement('.error');
?>

{% endhighlight %}

我们也有其他的命令去进行检测。需要说明的是所有的方法都以 `see` 作为前缀。

{% highlight php %}

<?php
$I->seeInCurrentUrl('/user/miles');
$I->seeCheckboxIsChecked('#agree');
$I->seeInField('user[name]', 'Miles');
$I->seeLink('Login');
?>

{% endhighlight %}

#### 条件断言

有些时候你希望断言失败后，不结束测试脚本。也许你有一个需要运行很久的测试脚本，并且你希望它能一直运行下去。 在这种情况下，你可以使用条件断言。 每一个 `see` 方法都有一个对应的 `canSee` 方法、 一个 `dontSee`方法 和一个 `cantSee` 方法。 

{% highlight php %}

<?php
$I->canSeeInCurrentUrl('/user/miles');
$I->canSeeCheckboxIsChecked('#agree');
$I->cantSeeInField('user[name]', 'Miles');
?>

{% endhighlight %}

每一个失败的断言都会显示在测试结果中。同时，失败的断言不会结束当前测试脚本。

#### 捕获

捕获命令在测试脚本检索页面中数据。想象一下，你的网站为每一个用户生成了密码，然后你想检查是否能通过该密码登陆到系统中去。

{% highlight php %}

<?php
$I->fillField('email', 'miles@davis.com')
$I->click('Generate Password');
$password = $I->grabTextFrom('#password');
$I->click('Login');
$I->fillField('email', 'miles@davis.com');
$I->fillField('password', $password);
$I->click('Log in!');
?>

{% endhighlight %}

捕获允许你通过命令在页面中获取值。

{% highlight php %}

<?php
$token = $I->grabTextFrom('.token');
$password = $I->grabTextFrom("descendant::input/descendant::*[@id = 'password']");
$api_key = $I->grabValueFrom('input[name=api]');
?>

{% endhighlight %}

#### 行为注释

在一个长的测试脚本中你应该描述你将要执行的动作，并且把结果记录下来。
`amGoingTo`, `expect`, `expectTo` 等命令会帮助你在测试中输出描述信息。

{% highlight php %}

<?php
$I->amGoingTo('submit user form with invalid values');
$I->fillField('user[email]', 'miles');
$I->click('Update');
$I->expect('the form is not submitted');
$I->see('Form is filled incorrectly');
?>

{% endhighlight %}

#### Cookies，Urls，标题，等等

cookie相关的方法：

{% highlight php %}

<?php
$I->setCookie('auth', '123345');
$I->grabCookie('auth');
$I->seeCookie('auth');
?>

{% endhighlight %}

检测页面标题的方法：

{% highlight php %}

<?php
$I->seeInTitle('Login');
$I->dontSeeInTitle('Register');
?>

{% endhighlight %}

url相关的方法：

{% highlight php %}

<?php
$I->seeCurrentUrlEquals('/login');
$I->seeCurrentUrlMatches('~$/users/(\d+)~');
$I->seeInCurrentUrl('user/1');
$user_id = $I->grabFromCurrentUrl('~$/user/(\d+)/~');
?>

{% endhighlight %}

## Selenium WebDriver

Codeception一个非常好的特性就是大部分的测试脚本可以很简单的移植后台运行。
我们有时候会想在真实的浏览器中运行测试脚本，Codeception支持这种特性，它支持在PhantomJs或Selenium WebDriver中去运行测试。

想要做到这一点，你只需要修改配置然后从新生成你的验收测试脚本，使用**WebDriver** 替代 PhpBrowser.

修改你的 `acceptance.suite.yml` 文件：

{% highlight yaml %}

class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver:
            url: {{your site url}}
            browser: firefox            
        - \Helper\Acceptance

{% endhighlight %}

想要使用Selenium 进行测试你需要[下载Selenium Server](http://seleniumhq.org/download/)然后运行 (或者你也可以使用 [PhantomJS](http://phantomjs.org/) 不需要使用 browser `ghostdriver` 模块)。

如果通过Selenium来运行验收测试脚本，FireFox浏览器会一步一步的运行所有的测试动作。

在这种情况下， `seeElement` 将不仅仅检查当前页面中是否存在某节点，同时它也会检测节点是否是真实可见的。

{% highlight php %}

<?php 
$I->seeElement('#modal'); 
?>

{% endhighlight %}


#### 等待

当测试web应用程序的时候，你可以需要等待JavaScript事件执行完成。因为JavaScript的异步机制，测试复杂的JavaScript交互程序会很困难。这就是为什么我们需要使用 `wait` 方法， 使用这个方法能指定javaScript事件执行多长时间。

例如：

{% highlight php %}

<?php
$I->waitForElement('#agree_button', 30); // secs
$I->click('#agree_button');
?>

{% endhighlight %}

在这个例子中，我们等待#agree_button按钮显示出来，然后点击它。如果我们不给予30秒钟的时间去显示，测试会失败。有很多其他的 `wait` 法你可以去使用。

点击 Codeception 的[WebDriver 模块文档](http://codeception.cn/docs/modules/WebDriver) 查看所有内容。

### Session 快照

进行测试时，常常需要去保存用户的session如果你需要为每个测试进行登入授权，你可以在每个测试开始时，填充登陆表单，进行登陆。运行这些步骤需要一些时间，特别是在Selenium中运行测试脚本时，运行时间会非常非常的慢。Codeception允许你在测试脚本中分享cookies，所以只需要登陆一次就能在其他的测试脚本中自动登陆。

下面的例子写了一个 `test_login` 函数，并且调用了该函数：

{% highlight php %}

<?php
function test_login($I)
{
     // if snapshot exists - skipping login
     if ($I->loadSessionSnapshot('login')) return;
     // logging in
     $I->amOnPage('/login');
     $I->fillField('name', 'jon');
     $I->fillField('password', '123345');
     $I->click('Login');
     // saving snapshot
     $I->saveSessionSnapshot('login');
}
// in test:
$I = new AcceptanceTester($scenario);
test_login($I);
?>

{% endhighlight %}

推荐把上面写的 `test_login` 函数放入到`AcceptanceTester` 对象中。

### 多 Session 测试 

Codeception允许你在执行动作的时候使用不同的session。在一些情况下，我们需要测试不同用户之间的实时消息，这时候你需要同时打开2个浏览器窗口进行测试。Codeception有非常聪明的方法去做这个事情。我们叫它为 **Friends**.

{% highlight php %}

<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('try multi session');
$I->amOnPage('/messages');
$nick = $I->haveFriend('nick');
$nick->does(function(AcceptanceTester $I) {
    $I->amOnPage('/messages/new');
    $I->fillField('body', 'Hello all!')
    $I->click('Send');
    $I->see('Hello all!', '.message');
});
$I->wait(3);
$I->see('Hello all!', '.message');
?>

{% endhighlight %}

这时候，我们在二个窗口中运行Friends对象的 `does` 方法。

### 数据清理

在测试时，你的测试脚本会改变网站中的数据。当你再次创建或者更新数据的时候就会出错。为了避免这个问题，你的数据库在每次开始测试之前必须被重载。 Codeception 提供一个 `Db` 模块进行重载。该模块每次完成测试之后会载入数据库初始化文件。为了保证重载正常工作，要创建一个包含原始数据的sql 文件，把文件放到 __/tests/_data__ 目录中。设置数据库连接和sql文件目录地址到Codeception全局配置中。

{% highlight yaml %}

# in codeception.yml:
modules:
    config:
        Db:
            dsn: '[set pdo dsn here]'
            user: '[set user]'
            password: '[set password]'
            dump: tests/_data/dump.sql

{% endhighlight %}

之后在`acceptance.suite.yml` 配置文件中把Db 模块打开。.

### 调试

Codeception 模块在测试运行的时候能打印变量的信息。只需要在执行测试脚本的时候加上 `--debug`参数。 使用`codecept_debug` 输出任意的数据。

{% highlight php %}

<?php
codecept_debug($I->grabTextFrom('#name'));
?>

{% endhighlight %}

如果出错，系统会将最后的错误快照存储到 __tests/_output__ 目录。 PhpBrowser 存储HTML代码，WebDriver存储页面快照。

有些时候你可能想要通过运行测试脚本来打开一个页面然后检测该页面。 在这种情况下你可以使用WebDriver模块的 [pauseExecution](http://codeception.cn/docs/modules/WebDriver#pauseExecution) 方法。

你可以记录测试脚本的所有测试步骤，然后一点一点的显示出来，具体可以查看[Recorder extension](http://codeception.cn/addons#CodeceptionExtensionRecorder). 

## 结束语

在Codeception中，写基于PhpBrowser的验收测试是一个很好的开始。你可以很容易的测试你的Joomla，Drupal，WordPress，或者基于框架开发的网站。编写验收测试就像是在描述测试人员的动作一样简单。它们可读性很高，并且编写非常简单。每次运行测试都记得要重载数据库哟。




* **下一章: [功能测试 >](/docs/04-FunctionalTests)**
* **上一章: [< 入门指南](/docs/02-GettingStarted)**
