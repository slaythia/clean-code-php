# clean-code-php

## 目录
  1. [介绍](#介绍)
  2. [变量](#变量)
  3. [函数](#函数)
  4. [对象和数据结构 Objects and Data Structures](#objects-and-data-structures)
  5. [类的SOLID原则 Classes](#classes)
     1. [S: 单一功能 Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
     2. [O: 开闭原则 Open/Closed Principle (OCP)](#openclosed-principle-ocp)
     3. [L: 里氏替换 Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
     4. [I: 接口隔离 Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     5. [D: 依赖反转 Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)

## 介绍
本文由 php-cpm 基于 yangweijie 的[clen php code](https://github.com/jupeter/clean-code-php)翻译并同步大量原文内容，欢迎大家指正。

本文参考自 Robert C. Martin的[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)  书中的软件工程师的原则
,适用于PHP。 这不是风格指南。 这是一个关于开发可读、可复用并且可重构的PHP软件指南。

并不是这里所有的原则都得遵循，甚至很少的能被普遍接受。 这些虽然只是指导，但是都是*Clean Code*作者多年总结出来的。

本文受到 [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript) 的启发

## **变量**
### 使用见字知意的变量名

**坏:**
```php
$ymdstr = $moment->format('y-m-d');
```

**好:**
```php
$currentDate = $moment->format('y-m-d');
```
**[⬆ 返回顶部](#目录)**

### 同一个实体要用相同的变量名

**坏:**
```php
getUserInfo();
getClientData();
getCustomerRecord();
```

**好:**
```php
getUserInfo();
getUserData();
getUserRecord();
```
**[⬆ 返回顶部](#目录)**

### 使用便于搜索的名称
写代码是用来读的。所以写出可读性高、便于搜索的代码至关重要。
命名变量时如果没有有意义、不好理解，那就是在伤害读者。
请让你的代码便于搜索。

**坏:**
```php
// What the heck is 86400 for?
addExpireAt(86400);

```

**好:**
```php
// Declare them as capitalized `const` globals.
interface DateGlobal {
    const SECONDS_IN_A_DAY = 86400;
}

addExpireAt(DateGlobal::SECONDS_IN_A_DAY);
```
**[⬆ 返回顶部](#目录)**


### 使用解释型变量
**坏:**
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/';
preg_match($cityZipCodeRegex, $address, $matches);
saveCityZipCode($matches[1], $matches[2]);
```

**不错:**

It's better, but we are still heavily dependent on regex.

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/';
preg_match($cityZipCodeRegex, $address, $matches);

list(, $city, $zipCode) = $matches;
saveCityZipCode($city, $zipCode);
```

**好:**

Decrease dependence on regex by naming subpatterns.
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,\\]+[,\\\s]+(?<city>.+?)\s*(?<zipCode>\d{5})?$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```

**[⬆ 返回顶部](#目录)**

### 少用无意义的变量名
别让读你的代码的人猜你写的变量是什么意思。
写清楚好过模糊不清。

**坏:**
```php
$l = ['Austin', 'New York', 'San Francisco'];
for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
  // 等等, `$li` 又代表什么?
  dispatch($li);
}
```

**好:**
```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
});
```
**[⬆ 返回顶部](#目录)**


### 不要添加不必要上下文
如果从你的类名、对象名已经可以得知一些信息，就别再在变量名里重复。

**坏:**

```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;

    //...
}
```

**好:**

```php
class Car
{
    public $make;
    public $model;
    public $color;

    //...
}
```
**[⬆ 返回顶部](#目录)**

### 合理使用参数默认值，没必要在方法里再做默认值检测
**坏:**
```php
function createMicrobrewery($name = null) {
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}

```

**好:**
```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.') {
    // ...
}

```
**[⬆ 返回顶部](#目录)**

## **函数**
### 函数参数（最好少于2个）
限制函数参数个数极其重要，这样测试你的函数容易点。有超过3个可选参数参数导致一个爆炸式组合增长，你会有成吨独立参数情形要测试。

无参数是理想情况。1个或2个都可以，最好避免3个。再多就需要加固了。通常如果你的函数有超过两个参数，说明他要处理的事太多了。 如果必须要传入很多数据，建议封装一个高级别对象作为参数。

**坏:**
```php
function createMenu($title, $body, $buttonText, $cancellable) {
    // ...
}
```

**好:**
```php
class MenuConfig
{
    public $title;
    public $body;
    public $buttonText;
    public $cancellable = false;
}

$config = new MenuConfig();
$config->title = 'Foo';
$config->body = 'Bar';
$config->buttonText = 'Baz';
$config->cancellable = true;

function createMenu(MenuConfig $config) {
  // ...
}

```
**[⬆ 返回顶部](#目录)**


### 函数应该只做一件事
这是迄今为止软件工程里最重要的一个规则。当一个函数做超过一件事的时候，他们就难于实现、测试和理解。当你把一个函数拆分到只剩一个功能时，他们就容易被重构，然后你的代码读起来就更清晰。如果你光遵循这条规则，你就领先于大多数开发者了。

**坏:**
```php
function emailClients($clients) {
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```

**好:**
```php
function emailClients($clients) {
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients($clients) {
    return array_filter($clients, 'isClientActive');
}

function isClientActive($client) {
    $clientRecord = $db->find($client);
    return $clientRecord->isActive();
}
```
**[⬆ 返回顶部](#目录)**

### 函数名应该是有意义的动词（或表明具体做了什么事）

**坏:**

```php
class Email
{
    //...

    public function handle()
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 啥？handle处理一个消息干嘛了？
$message->handle();
```

**好:**

```php
class Email 
{
    //...

    public function send()
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 简单明了
$message->send();
```

**[⬆ 返回顶部](#目录)**

### 函数应当只有一层抽象abstraction

当你抽象层次过多时时，函数处理的事情太多了。需要拆分功能来提高可重用性和易用性，以便简化测试。
（译者注：这里从示例代码看应该是指嵌套过多）

**坏:**

```php
function parseBetterJSAlternative($code)
{
    $regexes = [
        // ...
    ];
    
    $statements = split(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }
    
    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }
    
    foreach ($ast as $node) {
        // parse...
    }
}
```

**坏:**

我们把一些方法从循环中提取出来，但是`parseBetterJSAlternative()`方法还是很复杂，而且不利于测试。

```php
function tokenize($code)
{
    $regexes = [
        // ...
    ];
    
    $statements = split(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }
    
    return $tokens;
}

function lexer($tokens)
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }
    
    return $ast;
}

function parseBetterJSAlternative($code)
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // parse...
    }
}
```

**好:**

最好的解决方案是把 `parseBetterJSAlternative()`方法的依赖移除。

```php
class Tokenizer
{
    public function tokenize($code)
    {
        $regexes = [
            // ...
        ];

        $statements = split(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}
```

```php
class Lexer
{
    public function lexify($tokens)
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}
```

```php
class BetterJSAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse($code)
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // parse...
        }
    }
}
```

这样我们可以对依赖做mock，并测试`BetterJSAlternative::parse()`运行是否符合预期。

**[⬆ 返回顶部](#目录)**


### 不要用flag作为函数的参数
flag就是在告诉大家，这个方法里处理很多事。前面刚说过，一个函数应当只做一件事。 把不同flag的代码拆分到多个函数里。

**坏:**
```php
function createFile($name, $temp = false) {
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

**好:**
```php
function createFile($name) {
    touch($name);
}

function createTempFile($name) {
    touch('./temp/'.$name);
}
```
**[⬆ 返回顶部](#目录)**

### 避免副作用
一个函数做了比获取一个值然后返回另外一个值或值们会产生副作用如果。副作用可能是写入一个文件，修改某些全局变量或者偶然的把你全部的钱给了陌生人。

现在，你的确需要在一个程序或者场合里要有副作用，像之前的例子，你也许需要写一个文件。你想要做的是把你做这些的地方集中起来。不要用几个函数和类来写入一个特定的文件。用一个服务来做它，一个只有一个。

重点是避免常见陷阱比如对象间共享无结构的数据，使用可以写入任何的可变数据类型，不集中处理副作用发生的地方。如果你做了这些你就会比大多数程序员快乐。

**坏:**
```php
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName() {
    global $name;

    $name = preg_split('/ /', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```

**好:**
```php
function splitIntoFirstAndLastName($name) {
    return preg_split('/ /', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```
**[⬆ 返回顶部](#目录)**

### 不要写全局函数
在大多数语言中污染全局变量是一个坏的实践，因为你可能和其他类库冲突并且你api的用户不明白为什么直到他们获得产品的一个异常。让我们看一个例子：如果你想配置一个数组，你可能会写一个全局函数像`config()`，但是可能和试着做同样事的其他类库冲突。这就是为什么单例设计模式和简单配置会更好的原因。

**坏:**

```php
function config()
{
    return  [
        'foo' => 'bar',
    ]
}
```

**好:**

Create PHP configuration file or something else

```php
// config.php
return [
    'foo' => 'bar',
];
```

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get($key)
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```

Load configuration from file and create instance of `Configuration` class 

```php
$configuration = new Configuration($configuration);
```

And now you must use instance of `Configuration` in your application.


**[⬆ 返回顶部](#目录)**

### Don't use a Singleton pattern
Singleton is a [anti-pattern](https://en.wikipedia.org/wiki/Singleton_pattern).

**坏:**

```php
class DBConnection
{
    private static $instance;

    private function __construct($dsn)
    {
        // ...
    }

    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```

**好:**

```php
class DBConnection
{
    public function __construct(array $dsn)
    {
        // ...
    }

     // ...
}
```

Create instance of `DBConnection` class and configure it with [DSN](http://php.net/manual/en/pdo.construct.php#refsect1-pdo.construct-parameters).

```php
$connection = new DBConnection($dsn);
```

And now you must use instance of `DBConnection` in your application.

**[⬆ 返回顶部](#目录)**

### 封装条件语句

**坏:**

```php
if ($article->state === 'published') {
    // ...
}
```

**好:**

```php
if ($article->isPublished()) {
    // ...
}
```
**[⬆ 返回顶部](#目录)**

### 避免用反义条件判断

**坏:**
```php
function isDOMNodeNotPresent($node) {
    // ...
}

if (!isDOMNodeNotPresent($node)) {
    // ...
}
```

**好:**
```php
function isDOMNodePresent($node) {
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```
**[⬆ 返回顶部](#目录)**

### 避免条件声明
这看起来像一个不可能任务。当人们第一次听到这句话是都会这么说。
"没有一个`if声明`" 答案是你可以使用多态来达到许多case语句里的任务。第二个问题很常见， “那么为什么我要那么做？” 答案是前面我们学过的一个整洁代码原则：一个函数应当只做一件事。当你有类和函数有很多`if`声明,你自己知道你的函数做了不止一件事。记住，只做一件事。

**坏:**
```php
class Airplane {
    // ...
    public function getCruisingAltitude() {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

**好:**
```php
class Airplane {
    // ...
}

class Boeing777 extends Airplane {
    // ...
    public function getCruisingAltitude() {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne extends Airplane {
    // ...
    public function getCruisingAltitude() {
        return $this->getMaxAltitude();
    }
}

class Cessna extends Airplane {
    // ...
    public function getCruisingAltitude() {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```
**[⬆ 返回顶部](#目录)**

### 避免类型检查 (part 1)
PHP是弱类型的,这意味着你的函数可以接收任何类型的参数。
有时候你为这自由所痛苦并且在你的函数渐渐尝试类型检查。
有很多方法去避免这么做。第一种是统一API。

**坏:**

```php
function travelToTexas($vehicle)
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->peddleTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**好:**

```php
function travelToTexas(Traveler $vehicle)
{
    $vehicle->travelTo(new Location('texas'));
}
```
**[⬆ 返回顶部](#目录)**

### 避免类型检查 (part 2)
如果你正使用基本原始值比如字符串、整形和数组，要求版本是PHP 7+，不用多态，需要类型检测，
那你应当考虑[类型声明](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)或者严格模式。
提供了基于标准PHP语法的静态类型。 手动检查类型的问题是做好了需要好多的废话，好像为了安全就可以不顾损失可读性。
保持你的PHP 整洁，写好测试，做好代码回顾。做不到就用PHP严格类型声明和严格模式来确保安全。

**坏:**

```php
function combine($val1, $val2)
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**好:**

```php
function combine(int $val1, int $val2)
{
    return $val1 + $val2;
}
```
**[⬆ 返回顶部](#目录)**

### 移除僵尸代码
僵尸代码和重复代码一样坏。没有理由保留在你的代码库中。如果从来没被调用过，就删掉！
因为还在代码版本库里，因此很安全。

**坏:**
```php
function oldRequestModule($url) {
    // ...
}

function newRequestModule($url) {
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');

```

**好:**
```php
function requestModule($url) {
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```
**[⬆ 返回顶部](#目录)**


## **Objects and Data Structures**
### Use getters and setters
In PHP you can set `public`, `protected` and `private` keywords for methods. 
Using it, you can control properties modification on an object. 

* When you want to do more beyond getting an object property, you don't have
to look up and change every accessor in your codebase.
* Makes adding validation simple when doing a `set`.
* Encapsulates the internal representation.
* Easy to add logging and error handling when getting and setting.
* Inheriting this class, you can override default functionality.
* You can lazy load your object's properties, let's say getting it from a
server.

Additionally, this is part of Open/Closed principle, from object-oriented 
design principles.

**坏:**
```php
class BankAccount {
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

**好:**
```php
class BankAccount {
    private $balance;
    
    public function __construct($balance = 1000) {
      $this->balance = $balance;
    }
    
    public function withdrawBalance($amount) {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }
        $this->balance -= $amount;
    }
    
    public function depositBalance($amount) {
        $this->balance += $amount;
    }
    
    public function getBalance() {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdrawBalance($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();

```
**[⬆ 返回顶部](#目录)**


### Make objects have private/protected members

**坏:**
```php
class Employee {
    public $name;
    
    public function __construct($name) {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```

**好:**
```php
class Employee {
    protected $name;
    
    public function __construct($name) {
        $this->name = $name;
    }
    
    public function getName() {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```
**[⬆ 返回顶部](#目录)**


## **Classes**

### Single Responsibility Principle (SRP)
As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimizing the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify a piece of it,
it can be difficult to understand how that will affect other dependent modules in
your codebase.

**坏:**
```php
class UserSettings {
    private $user;

    public function __construct($user) {
        $this->user = $user;
    }
    
    public function changeSettings($settings) {
        if ($this->verifyCredentials()) {
            // ...
        }
    }
    
    private function verifyCredentials() {
        // ...
    }
}
```

**好:**
```php
class UserAuth {
    private $user;

    public function __construct($user) {
        $this->user = $user;
    }
    
    public function verifyCredentials() {
        // ...
    }
}


class UserSettings {
    private $user;

    public function __construct($user) {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }
    
    public function changeSettings($settings) {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```
**[⬆ 返回顶部](#目录)**

### Open/Closed Principle (OCP)

As stated by Bertrand Meyer, "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

**坏:**

```php
abstract class Adapter
{
    protected $name;

    public function getName()
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();
        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();
        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct($adapter)
    {
        $this->adapter = $adapter;
    }
    
    public function fetch($url)
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }
    
    protected function makeAjaxCall($url)
    {
        // request and return promise
    }
    
    protected function makeHttpCall($url)
    {
        // request and return promise
    }
}
```

**好:**

```php
interface Adapter
{
    public function request($url);
}

class AjaxAdapter implements Adapter
{
    public function request($url)
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request($url)
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }
    
    public function fetch($url)
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆ 返回顶部](#目录)**

### Liskov Substitution Principle (LSP)

This is a scary term for a very simple concept. It's formally defined as "If S
is a subtype of T, then objects of type T may be replaced with objects of type S
(i.e., objects of type S may substitute objects of type T) without altering any
of the desirable properties of that program (correctness, task performed,
etc.)." That's an even scarier definition.

The best explanation for this is if you have a parent class and a child class,
then the base class and child class can be used interchangeably without getting
incorrect results. This might still be confusing, so let's take a look at the
classic Square-Rectangle example. Mathematically, a square is a rectangle, but
if you model it using the "is-a" relationship via inheritance, you quickly
get into trouble.

**坏:**
```php
class Rectangle
{
    protected $width;
    protected $height;

    public function __construct()
    {
        $this->width = 0;
        $this->height = 0;
    }

    public function render($area)
    {
        // ...
    }

    public function setWidth($width)
    {
        $this->width = $width;
    }

    public function setHeight($height)
    {
        $this->height = $height;
    }

    public function getArea()
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth($width)
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(height)
    {
        $this->width = $this->height = $height;
    }
}

function renderLargeRectangles($rectangles)
{
    foreach ($rectangles as $rectangle) {
        $rectangle->setWidth(4);
        $rectangle->setHeight(5);
        $area = $rectangle->getArea(); // BAD: Will return 25 for Square. Should be 20.
        $rectangle->render($area);
    }
}

$rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles($rectangles);
```

**好:**
```php
abstract class Shape
{
    protected $width;
    protected $height;

    abstract public function getArea();

    public function render($area)
    {
        // ...
    }
}

class Rectangle extends Shape
{
    public function __construct()
    {
        parent::__construct();
        $this->width = 0;
        $this->height = 0;
    }

    public function setWidth($width)
    {
        $this->width = $width;
    }

    public function setHeight($height)
    {
        $this->height = $height;
    }

    public function getArea()
    {
        return $this->width * $this->height;
    }
}

class Square extends Shape
{
    public function __construct()
    {
        parent::__construct();
        $this->length = 0;
    }

    public function setLength($length)
    {
        $this->length = $length;
    }

    public function getArea()
    {
        return pow($this->length, 2);
    }
}

function renderLargeRectangles($rectangles)
{
    foreach ($rectangles as $rectangle) {
        if ($rectangle instanceof Square) {
            $rectangle->setLength(5);
        } elseif ($rectangle instanceof Rectangle) {
            $rectangle->setWidth(4);
            $rectangle->setHeight(5);
        }
        
        $area = $rectangle->getArea(); 
        $rectangle->render($area);
    }
}

$shapes = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles($shapes);
```
**[⬆ 返回顶部](#目录)**

### Interface Segregation Principle (ISP)
ISP states that "Clients should not be forced to depend upon interfaces that
they do not use." 

A good example to look at that demonstrates this principle is for
classes that require large settings objects. Not requiring clients to setup
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a "fat interface".

**坏:**
```php
interface WorkerInterface {
    public function work();
    public function eat();
}

class Worker implements WorkerInterface {
    public function work() {
        // ....working
    }
    public function eat() {
        // ...... eating in launch break
    }
}

class SuperWorker implements WorkerInterface {
    public function work() {
        //.... working much more
    }

    public function eat() {
        //.... eating in launch break
    }
}

class Manager {
  /** @var WorkerInterface $worker **/
  private $worker;
  
  public function setWorker(WorkerInterface $worker) {
        $this->worker = $worker;
    }

    public function manage() {
        $this->worker->work();
    }
}
```

**好:**
```php
interface WorkerInterface extends FeedableInterface, WorkableInterface {
}

interface WorkableInterface {
    public function work();
}

interface FeedableInterface {
    public function eat();
}

class Worker implements WorkableInterface, FeedableInterface {
    public function work() {
        // ....working
    }

    public function eat() {
        //.... eating in launch break
    }
}

class Robot implements WorkableInterface {
    public function work() {
        // ....working
    }
}

class SuperWorker implements WorkerInterface  {
    public function work() {
        //.... working much more
    }

    public function eat() {
        //.... eating in launch break
    }
}

class Manager {
  /** @var $worker WorkableInterface **/
    private $worker;

    public function setWorker(WorkableInterface $w) {
      $this->worker = $w;
    }

    public function manage() {
        $this->worker->work();
    }
}
```
**[⬆ 返回顶部](#目录)**

### Dependency Inversion Principle (DIP)
This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

This can be hard to understand at first, but if you've worked with PHP frameworks (like Symfony), you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

**坏:**
```php
class Worker {
  public function work() {
    // ....working
  }
}

class Manager {
    /** @var Worker $worker **/
    private $worker;
    
    public function __construct(Worker $worker) {
        $this->worker = $worker;
    }
    
    public function manage() {
        $this->worker->work();
    }
}

class SuperWorker extends Worker {
    public function work() {
        //.... working much more
    }
}
```

**好:**
```php
interface WorkerInterface {
    public function work();
}

class Worker implements WorkerInterface {
    public function work() {
        // ....working
    }
}

class SuperWorker implements WorkerInterface {
    public function work() {
        //.... working much more
    }
}

class Manager {
    /** @var WorkerInterface $worker **/
    private $worker;
    
    public function __construct(WorkerInterface $worker) {
        $this->worker = $worker;
    }
    
    public function manage() {
        $this->worker->work();
    }
}

```
**[⬆ 返回顶部](#目录)**

### Use method chaining
This pattern is very useful and commonly used it many libraries such
as PHPUnit and Doctrine. It allows your code to be expressive, and less verbose.
For that reason, I say, use method chaining and take a look at how clean your code
will be. In your class functions, simply return `this` at the end of every function,
and you can chain further class methods onto it.

**坏:**
```php
class Car {
    private $make, $model, $color;
    
    public function __construct() {
        $this->make = 'Honda';
        $this->model = 'Accord';
        $this->color = 'white';
    }
    
    public function setMake($make) {
        $this->make = $make;
    }
    
    public function setModel($model) {
        $this->model = $model;
    }
    
    public function setColor($color) {
        $this->color = $color;
    }
    
    public function dump() {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
```

**好:**
```php
class Car {
    private $make, $model, $color;
    
    public function __construct() {
        $this->make = 'Honda';
        $this->model = 'Accord';
        $this->color = 'white';
    }
    
    public function setMake($make) {
        $this->make = $make;
        
        // NOTE: Returning this for chaining
        return $this;
    }
    
    public function setModel($model) {
        $this->model = $model;
        
        // NOTE: Returning this for chaining
        return $this;
    }
    
    public function setColor($color) {
        $this->color = $color;
        
        // NOTE: Returning this for chaining
        return $this;
    }
    
    public function dump() {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
  ->setColor('pink')
  ->setMake('Ford')
  ->setModel('F-150')
  ->dump();
```
**[⬆ 返回顶部](#目录)**

### Prefer composition over inheritance
As stated famously in [*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four,
you should prefer composition over inheritance where you can. There are lots of
good reasons to use inheritance and lots of good reasons to use composition.
The main point for this maxim is that if your mind instinctively goes for
inheritance, try to think if composition could model your problem better. In some
cases it can.

You might be wondering then, "when should I use inheritance?" It
depends on your problem at hand, but this is a decent list of when inheritance
makes more sense than composition:

1. Your inheritance represents an "is-a" relationship and not a "has-a"
relationship (Human->Animal vs. User->UserDetails).
2. You can reuse code from the base classes (Humans can move like all animals).
3. You want to make global changes to derived classes by changing a base class.
(Change the caloric expenditure of all animals when they move).

**坏:**
```php
class Employee {
    private $name, $email;
    
    public function __construct($name, $email) {
        $this->name = $name;
        $this->email = $email;
    }
    
    // ...
}

// Bad because Employees "have" tax data. 
// EmployeeTaxData is not a type of Employee

class EmployeeTaxData extends Employee {
    private $ssn, $salary;
    
    public function __construct($name, $email, $ssn, $salary) {
        parent::__construct($name, $email);
        $this->ssn = $ssn;
        $this->salary = $salary;
    }
    
    // ...
}
```

**好:**
```php
class EmployeeTaxData {
    private $ssn, $salary;
    
    public function __construct($ssn, $salary) {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }
    
    // ...
}

class Employee {
    private $name, $email, $taxData;
    
    public function __construct($name, $email) {
        $this->name = $name;
        $this->email = $email;
    }
    
    public function setTaxData($ssn, $salary) {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }
    // ...
}
```
**[⬆ 返回顶部](#目录)**

## 别写重复代码 (DRY)

Try to observe the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle.

Do your absolute best to avoid duplicate code. Duplicate code is bad because 
it means that there's more than one place to alter something if you need to 
change some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your 
tomatoes, onions, garlic, spices, etc. If you have multiple lists that
you keep this on, then all have to be updated when you serve a dish with
tomatoes in them. If you only have one list, there's only one place to update!

Oftentimes you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing 
duplicate code means creating an abstraction that can handle this set of different 
things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the
SOLID principles laid out in the [Classes](#classes) section. Bad abstractions can be
worse than duplicate code, so be careful! Having said this, if you can make
a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself 
updating multiple places anytime you want to change one thing.

**Bad:**

```php
function showDeveloperList($developers)
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];
        
        render($data);
    }
}

function showManagerList($managers)
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];
        
        render($data);
    }
}
```

**Good:**

```php
function showList($employees)
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];
        
        render($data);
    }
}
```

**Very good:**

It is better to use a compact version of the code.

```php
function showList($employees)
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```
**[⬆ 返回顶部](#目录)**