---
layout: doc
title: 05-单元测试 - Codeception - 中文文档
---


# Unit Tests

Codeception 使用`PHPUnit` 测试框架来运行单元测试脚本。 因此，所有PHPUnit测试脚本都能在Codeception中运行。如果你曾经编写过PHPUnit测试脚本，那你现在可以跟以前一样去编写单元测试脚本。 Codeception添加了一些有用的功能去使普通任务更加简单。

关于单元测试的基本信息都从这里跳过，相应的我们将会告诉你Codeception为单元测试添加了哪些特性。

__再说一次：你不需要安装PHPUnit同运行单元测试脚本，Codeception能运行它们。__

## 创建测试脚本

Codeception拥有一些生成器让你很简单的生成测试脚本。你可以从生成一个传统的、基于 `\PHPUnit_Framework_TestCase`类的单元测试脚本开始。
使用下面的命令生成：

{% highlight bash %}

$ php codecept.phar generate:phpunit unit Example

{% endhighlight %}

Codeception拥有自己的单元测试脚本规范。我们需要另外一个命令去生成以Codeception为驱动的单元测试脚本。

{% highlight bash %}

$ php codecept.phar generate:test unit Example

{% endhighlight %}

这2个生成的测试脚本文件名称都为`ExampleTest`，存储在`tests/unit` 目录。

由`generate:test` 命令生成的测试脚本跟下面的代码类似：

{% highlight php %}

<?php
use Codeception\Util\Stub;

class ExampleTest extends \Codeception\TestCase\Test
{
   /**
    * @var UnitTester
    */
    protected $tester;

    // executed before each test
    protected function _before()
    {
    }

    // executed after each test
    protected function _after()
    {
    }
}
?>

{% endhighlight %}

这个类首先预定义了`_before` and `_after` 方法。 你可以在每个测试开始之前使用它们去创建一个测试对象，然后销毁它们。

正如你所见，它们跟PHPUnit不一样， `setUp` 和 `tearDown`方法被替换为： `_before`, `_after`。

真正的`setUp` and `tearDown` 会被父类 `\Codeception\TestCase\Test`实现， and set up the UnitTester class to have all the cool actions from Cept-files to be run as a part of unit tests. Just like in acceptance and functional tests you can choose the proper modules for `UnitTester` class in `unit.suite.yml` configuration file.


{% highlight yaml %}

# Codeception测试组件配置

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: 
        - Asserts
        - \Helper\Unit

{% endhighlight %}

### 传统的单元测试

在Codeception中编写单元测试与PHPUnit一样：

{% highlight php %}

<?php
class UserTest extends \Codeception\TestCase\Test
{
    public function testValidation()
    {
        $user = User::create();

        $user->username = null;
        $this->assertFalse($user->validate(['username']));

        $user->username = 'toolooooongnaaaaaaameeee';
        $this->assertFalse($user->validate(['username']));

        $user->username = 'davert';
        $this->assertTrue($user->validate(['username']));           
    }
}
?>

{% endhighlight %}

### 行为驱动开发(BDD)测试脚本规范

在为你的应用程序编写测试脚本的时候，你应该时刻准备它们会变化。测试脚本应该易读并且易于维护。如果你的应用程序的功能被修改，那你的测试脚本也要随之修改。 如果你的团队没有应用规范，那以后需要修复因为框架更新导致的问题。

不仅仅为应用程序编写单元测试是很重要的，让单元测试脚本可读性强也很重要。 我们为场景驱动验收测试和功能测试应用规范，也应该为单元测试和聚合测试应用规范。

针对这种情况，我们有一个单独的项目[Specify](https://github.com/Codeception/Specify) (可以下载phar包) 去方便编写符合规范的单元测试。

{% highlight php %}

<?php
class UserTest extends \Codeception\TestCase\Test
{
    use \Codeception\Specify;

    private $user;

    public function testValidation()
    {
        $this->user = User::create();

        $this->specify("username is required", function() {
            $this->user->username = null;
            $this->assertFalse($this->user->validate(['username']));
        });

        $this->specify("username is too long", function() {
            $this->user->username = 'toolooooongnaaaaaaameeee';
            $this->assertFalse($this->user->validate(['username']));
        });

        $this->specify("username is ok", function() {
            $this->user->username = 'davert';
            $this->assertTrue($this->user->validate(['username']));
        });
    }
}
?>        

{% endhighlight %}

使用 `specify` 代码块你能描述任意测试代码。 它让测试更加干净，并且让团队成员更容易理解测试代码。

 `specify` 代码块中的代码是独立的。 在上面的例子中对`$this->user` (作为任何其他对象属性)的修改，都不会影响其他的代码块。

同时你也可以添加[Codeception\Verify](https://github.com/Codeception/Verify)方便编写行为驱动类型的断言程序。它是一个微小的库，添加了更多阅读性高的断言函数，运行速度也很快，特别是你一直疑惑哪个参数 if you are always confused of which argument in `assert` calls is expected and which one is actual.

{% highlight php %}

<?php
verify($user->getName())->equals('john');
?>

{% endhighlight %}

## Using Modules

As in scenario-driven functional or acceptance tests you can access Actor class methods. If you write integration tests, it may be useful to include `Db` module for database testing. 

{% highlight yaml %}

# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: 
        - Asserts
        - Db
        - \Helper\Unit

{% endhighlight %}

To access UnitTester methods you can use `UnitTester` property in a test.

### Testing Database

Let's see how you can do some database testing:

{% highlight php %}

<?php
function testSavingUser()
{
    $user = new User();
    $user->setName('Miles');
    $user->setSurname('Davis');
    $user->save();
    $this->assertEquals('Miles Davis', $user->getFullName());
    $this->tester->seeInDatabase('users', array('name' => 'Miles', 'surname' => 'Davis'));
}
?>

{% endhighlight %}

To enable the database functionality in the unit tests please make sure the `Db` module is part of the enabled module list in the unit.suite.yml configuration file. 
The database will be cleaned and populated after each test, as it happens for acceptance and functional tests.
If it's not your required behavior, please change the settings of `Db` module for the current suite.

### Interacting with the Framework

Probably you should not access your database directly if your project already uses ORM for database interactions.
Why not use the ORM directly inside your tests? Let's try to write a test using Laravel's ORM Eloquent, for this we need configured Laravel5 module. We won't need its web interaction methods like `amOnPage` or `see`, so let's enable only ORM part of it:

{% highlight yaml %}

class_name: UnitTester
modules:
    enabled:
        - Asserts
        - Laravel5:
            part: ORM
        - \Helper\Unit

{% endhighlight %}

We included Laravel5 module as we did for functional testing. Let's see how we can use it for integration tests:

{% highlight php %}

<?php
function testUserNameCanBeChanged()
{
    // create a user from framework, user will be deleted after the test
    $id = $this->tester->haveRecord('users', ['name' => 'miles']);
    // access model
    $user = User::find($id);
    $user->setName('bill');
    $user->save();
    $this->assertEquals('bill', $user->getName());
    // verify data was saved using framework methods
    $this->tester->seeRecord('users', ['name' => 'bill']);
    $this->tester->dontSeeRecord('users', ['name' => 'miles']);
}
?>

{% endhighlight %}

The very similar approach can be used to all frameworks that have ORM implementing ActiveRecord pattern.
These are Yii2 and Phalcon, they have methods `haveRecord`, `seeRecord`, `dontSeeRecord` working in the same manner. They also should be included with specifying `part: ORM` in order to not use functional testing actions.

In case you are using Symfony2 with Doctrine you may not enable Symfony2 itself but use only Doctrine2 only:

{% highlight yaml %}

class_name: UnitTester
modules:
    enabled:
        - Asserts
        - Doctrine2:
            depends: Symfony2
        - \Helper\Unit

{% endhighlight %}

In this case you can use methods from Doctrine2 module, while Doctrine itself uses Symfony2 module to establish connection to database. In this case a test may look like:

{% highlight php %}

<?php
function testUserNameCanBeChanged()
{
    // create a user from framework, user will be deleted after the test
    $id = $this->tester->haveInRepository('Acme\DemoBundle\Entity\User', ['name' => 'miles']);
    // get entity manager by accessing module
    $em = $this->getModule('Doctrine2')->em;
    // get real user
    $user = $em->find('Acme\DemoBundle\Entity\User', $id);
    $user->setName('bill');
    $em->persist($user);
    $em->flush();
    $this->assertEquals('bill', $user->getName());
    // verify data was saved using framework methods
    $this->tester->seeInRepository('Acme\DemoBundle\Entity\User', ['name' => 'bill']);
    $this->tester->dontSeeInRepository('Acme\DemoBundle\Entity\User', ['name' => 'miles']);
}
?>

{% endhighlight %}

In both examples you should not be worried about the data persistence between tests.
Doctrine2 module as well as Laravel4 module will clean up created data at the end of a test. 
This is done by wrapping a test in a transaction and rolling it back afterwards. 

### Accessing Module

Codeception allows you to access properties and methods of all modules defined for this suite. Unlike using the UnitTester class for this purpose, using module directly grants you access to all public properties of that module.

We already demonstrated this case in previous code piece where we accessed Entity Manager from a Doctrine2 module

{% highlight php %}

<?php
/** @var Doctrine\ORM\EntityManager */
$em = $this->getModule('Doctrine2')->em;
?>

{% endhighlight %}

If you use `Symfony2` module, here is the way you can access Symfony container:

{% highlight php %}

<?php
/** @var Symfony\Component\DependencyInjection\Container */
$container = $this->getModule('Symfony2')->container;
?>

{% endhighlight %}

The same can be done for all public properties of an enabled module. Accessible properties are listed in the module reference

### Cest

Alternatively to testcases extended from `PHPUnit_Framework_TestCase` you may use Codeception-specific Cest format. It does not require to be extended from any other class. All public methods of this class are tests.

The example above can be rewritten in scenario-driven manner like this:

{% highlight php %}

<?php
class UserCest
{
    public function validateUser(UnitTester $t)
    {
        $user = $t->createUser();
        $user->username = null;
        $t->assertFalse($user->validate(['username']); 

        $user->username = 'toolooooongnaaaaaaameeee';
        $t->assertFalse($user->validate(['username']));

        $user->username = 'davert';
        $t->assertTrue($user->validate(['username']));

        $t->seeInDatabase('users', ['name' => 'Miles', 'surname' => 'Davis']);
    }
}
?>

{% endhighlight %}

For unit testing you may include `Asserts` module, that adds regular assertions to UnitTester which you may access from `$t` variable.

{% highlight yaml %}

# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: 
        - Asserts
        - Db
        - \Helper\Unit

{% endhighlight %}

[Learn more about Cest format](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes).

<div class="alert alert-info">
It may look like Cest format is too simple for writing tests. It doesn't provide assertion methods,
methods to create mocks and stubs or even accessing the module with `getModule`, as we did in example above.
However Cest format is better at separating concerns. Test code does not interfere with support code, provided by `UnitTester` object. All additional actions you may need in your unit/integration tests you can implement in `Helper\Unit` class. This is the recommended approach, and allows keeping tests verbose and clean.
</div>


### Stubs

Codeception provides a tiny wrapper over PHPUnit mocking framework to create stubs easily. Include `\Codeception\Util\Stub` to start creating dummy objects.

In this example we instantiate object without calling a constructor and replace `getName` method to return value *john*.

{% highlight php %}

<?php
$user = Stub::make('User', ['getName' => 'john']);
$name = $user->getName(); // 'john'
?>

{% endhighlight %}

Stubs are created with PHPUnit's mocking framework. Alternatively you can use [Mockery](https://github.com/padraic/mockery) (with [Mockery module](https://github.com/Codeception/MockeryModule)), [AspectMock](https://github.com/Codeception/AspectMock) or others.

Full reference on Stub util class can be found [here](/docs/reference/Stub).

## Conclusion

PHPUnit tests are first-class citizens in test suites. Whenever you need to write and execute unit tests, you don't need to install PHPUnit, but use Codeception to execute them. Some nice features can be added to common unit tests by integrating Codeception modules. For the most of unit and integration testing PHPUnit tests are just enough. They are fast and easy to maintain.




* **Next Chapter: [ReusingTestCode >](/docs/06-ReusingTestCode)**
* **Previous Chapter: [< FunctionalTests](/docs/04-FunctionalTests)**
