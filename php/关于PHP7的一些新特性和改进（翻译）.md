# 关于PHP7的一些新特性和改进（翻译）

## PHP7.0

## 新特性

### Spaceship Operator
    var_dump(1 <=> 2) //int(-1)
    var_dump(2 <=> 1) //int(0)
    var_dump(1 <=> 1) //int(1)
    
* 只能两元素进行比较，不支持多个操作符一起使用
* 不支持对象比较
* 比较级和`==, !=, ===, !==`同级

### Null Coalesce Operator
    //< 7.0
    $param = isset($GET['id']) ? $_GET['id'] : 1;
    // >= 7.0
    $param =  $_GET['id'] ?? 1;
    
### Strict Type Declarations(严格类型)
    //declare(strict_types=1);
    function sumOfInts(int ...$ints)
    {
        return array_sum($ints);
    }
    var_dump(sumOfInts(2, '3', 4.1)); // int(9)
    
* PHP7类型定义分为两种模式： **coercive**(强制转换，默认)和**strict**(严格)
* **coercive**下函数定义和传参没有要求，当传入的参数类型与定义不同，会进行强制类型转换
* **strict**类型使用要在文件开始声明
* **strict**类型对函数参数定义没做要求，但对传入的参数有要求。如果参数定义了int类型，传入了float，会抛出TypeError异常
*  `int float string bool`等现在不能作为类名或函数名定义

### Return Type Declarations
    function arraySum(array ...$arrays) : array
    {...}

* supported: `string, int, float, bool, array, callable, self, parent, Closure, className, functionName`
* 其中`self, parent`只有方法可以返回
* 返回类型必须要严格，特别是在继承关系中

        class A {}
        class B extends A {}
        
        
        class C
        {
            public function test() : A
            {
                return new A;
            }
        }
    
        class D extends C
        {
            // overriding method C::test() : A
            public function test() : B // cause E_COMPILE_ERROR
            {
                return new B;
            }
        }  
        
### Anonymous Classes(匿名类)
    $util->setLogger(new class($msg) {
        private $msg;
        public function __construct($msg) {
            $this->msg = $msg;
        }
        public function log($msg)
        {
            echo $msg;
        }
    });
    
* 匿名函数嵌套时，要想使用外部类的私有或保护属性，则必须继承外部类，并且属性要通过构造方法传入

### Unicode Codepoint Escape Syntax
    echo "\u{aa}"; // ª
    echo "\u{0000aa}"; // ª (same as before but with optional leading 0's)
    echo "\u{9999}"; // 香
    
### Closure call() Method
将一个闭包和一个类的定义联系在一起

    class A {private $x = 1;}
    
    // Pre PHP 7 code
    $getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // intermediate closure
echo $getX(); // 1
    // PHP 7+ codeO
    $getX = function() {return $this->x;};
echo $getX->call(new A); // 1

### Filtered unserialize()
反序列化类，添加白名单机制

    // converts all objects into __PHP_Incomplete_Class object
    $data = unserialize($foo, ["allowed_classes" => false]);
    
    // converts all objects into __PHP_Incomplete_Class object except those of MyClass and MyClass2
    $data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);
    
    // default behaviour (same as omitting the second argument) that accepts all classes
    $data = unserialize($foo, ["allowed_classes" => true]);
    
### IntlChar Class
新增加的 IntlChar类旨在暴露出更多的ICU功能。这个类自身定义了许多静态方法用于操作多字符集的 unicode 字符。（ICU？）

    printf('%x', IntlChar::CODEPOINT_MAX); // 10ffff
    echo IntlChar::charName('@'); // COMMERCIAL AT
    var_dump(IntlChar::ispunct('!')); // bool(true)
    
*ps:必须装Intl扩展,全局命名空间的函数名不能为IntlChar*

### Expectations
`assert()`函数现在支持当断言为false是抛出一个错误类

    ini_set('assert.exception', 1);
    class CustomError extends AssertionError {}
    assert(false, new CustomError('Some error message'));
    
*ps:需要打开assert,exception配置，默认关闭以兼容老代码*

### Group use Declarations

    // Pre PHP 7 code
    use some\namespace\ClassA;
    use some\namespace\ClassB;
    use some\namespace\ClassC as C;
    
    // PHP 7+ code
    use some\namespace\{ClassA, ClassB, ClassC as C};
    
### Integer Division with intdiv()（整除）

    var_dump(intdiv(10, 3)); // int(3)
    
### session_start() Options
允许在`session_start`函数中传入数组，来改变php.ini中session的初始参数

    session_start(['cache_limiter' => 'private']); 
    
### CSPRNG Functions
生成随机字符串和整数的两个函数（如果没有足够的随机数生成，会抛出异常）

    string random_bytes(int length);
    int random_int(int min, int max);
    
### Support for Array Constants in define()
支持`define`定义常量值为数组

    define('ALLOWED_IMAGE_EXTENSIONS', ['jpg', 'jpeg', 'gif', 'png']);
    
### preg_replace_callback_array() Function
使自定义的正则替换函数(`preg_replace_callback`)代码变得紧凑

    $tokenStream = []; // [tokenName, lexeme] pairs
    $input = <<<'end'
    $a = 3; // variable initialisation
    end;
    
    // Pre PHP 7 code
    preg_replace_callback(
        [
            '~\$[a-z_][a-z\d_]*~i',
            '~=~',
            '~[\d]+~',
            '~;~',
            '~//.*~'
        ],
        function ($match) use (&$tokenStream) {
            if (strpos($match[0], '$') === 0) {
                $tokenStream[] = ['T_VARIABLE', $match[0]];
            } elseif (strpos($match[0], '=') === 0) {
                $tokenStream[] = ['T_ASSIGN', $match[0]];
            } elseif (ctype_digit($match[0])) {
                $tokenStream[] = ['T_NUM', $match[0]];
            } elseif (strpos($match[0], ';') === 0) {
                $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
            } elseif (strpos($match[0], '//') === 0) {
                $tokenStream[] = ['T_COMMENT', $match[0]];
            }
        },
        $input
    );
    
    // PHP 7+ code
    preg_replace_callback_array(
        [
            '~\$[a-z_][a-z\d_]*~i' => function ($match) use (&$tokenStream) {
                $tokenStream[] = ['T_VARIABLE', $match[0]];
            },
            '~=~' => function ($match) use (&$tokenStream) {
                $tokenStream[] = ['T_ASSIGN', $match[0]];
            },
            '~[\d]+~' => function ($match) use (&$tokenStream) {
                $tokenStream[] = ['T_NUM', $match[0]];
            },
            '~;~' => function ($match) use (&$tokenStream) {
                $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
            },
            '~//.*~' => function ($match) use (&$tokenStream) {
                $tokenStream[] = ['T_COMMENT', $match[0]];
            }
        ],
        $input
    );
    
### Reflection Additions
增加新的反射类

* `ReflectionGenerator` - generators的反射机制
* `ReflectionType `     - 更好支持类型定义

### Generator Return Expressions
生成器定义支持使用`return`,并在`yield`结束之后使用return值

    $gen = (function() {
        yield 1;
        yield 2;
    
        return 3;
    })();
    
    foreach ($gen as $val) {
        echo $val, PHP_EOL;
    }
    
    echo $gen->getReturn(), PHP_EOL;
    
    // output:
    // 1
    // 2
    // 3
    
### Generator Delegation
生成器允许返回生成器

    function gen()
    {
        yield 1;
        yield 2;
    
        return yield from gen2();
    }
    
    function gen2()
    {
        yield 3;
    
        return 4;
    }
    
    $gen = gen();
    
    foreach ($gen as $val)
    {
        echo $val, PHP_EOL;
    }
    
    echo $gen->getReturn();
    
    // output
    // 1
    // 2
    // 3
    // 4
    
## 改进

### Loosening Reserved Word Restrictions
保留字现在可以作为`class,interface,trait`的属性，常量和方法名`class`除外

    // 'new', 'private', and 'for' were previously unusable
    Project::new('ProjectName')->private()->for('purposehere')->with('username here');
    
### Uniform Variable Syntax(链式调用改变)

                        // old meaning         // new meaning
    $$foo['bar']['baz'] ${$foo['bar']['baz']} ($$foo)['bar']['baz']
    $foo->$bar['baz']   $foo->{$bar['baz']}   ($foo->$bar)['baz']
$foo->$bar['baz']() $foo->{$bar['baz']}() ($foo->$bar)['baz']()
Foo::$bar['baz']()  Foo::{$bar['baz']}()  (Foo::$bar)['baz']()

### Exceptions in the Engine
`Exception`类的修改,添加了`Error`类

    interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
            
### Throwable Interface
添加`interface Throwable`

### Integer Semantics

    // Was: int(-9223372036854775808)
    // Now: int(0)
    var_dump((int)NAN);
    // Was: int(-9223372036854775808)
    // Now: int(0)
    var_dump((int)INF);
    // Was: int(4611686018427387904)
    // Now: bool(false) and E_WARNING
    var_dump(1 << -2);
    // Was: int(8)
    // Now: int(0)
    
* `NAN`和`INF`为0
* 位移超过位长会返回0,-1或者报错

### JSON Extension Replaced with JSOND
json解释器换成jsond

### ZPP Failure on Overflow
以前当函数参数要求为int时传入int参数，一般会发生强制类型转换，如果传入的float值太大，会发生截断。现在，但发生类型转换的时候，如果失败会返回null或E_WARNING。

### Fixes to foreach()'s Behaviour
* `current()`现在不影响foreach迭代的指针
* `unset`等操作会等到迭代完成后执行

### Changes to list()'s Behaviour
list赋值顺序改变

    $a = [1, 2];
    list($a, $b) = $a;
    // OLD: $a = 1, $b = 2
    // NEW: $a = 1, $b = null + "Undefined index 1"
    
    $b = [1, 2];
    list($a, $b) = $b;
    // OLD: $a = null + "Undefined index 0", $b = 2
    // NEW: $a = 1, $b = 2
    
### Changes to Division by Zero Semantics

    var_dump(3/0); // float(INF) + E_WARNING
    var_dump(0/0); // float(NAN) + E_WARNING
    
    var_dump(0%0); // DivisionByZeroError
    
    intdiv(PHP_INT_MIN, -1); // ArithmeticError
    
### Fixes to Custom Session Handler Return Values

    class FileSessionHandler implements SessionHandlerInterface
    {
        private $savePath;
        function open($savePath, $sessionName)
        {
            return false; // always fail
        }

        function close(){return true;}
        function read($id){}
        function write($id, $data){}
        function destroy($id){}
        function gc($maxlifetime){}
    }
    
    session_set_save_handler(new FileSessionHandler());
    session_start(); // doesn't cause an error in pre PHP 7 code
    
### Deprecation of PHP 4-Style Constructors

    // Namespaced classes do not recognize PHP 4 constructors
    namespace NS;
    class Filter {
     
        // Not a constructor
        function filter($a) {}
    }
    // Defining __construct and filter makes filter a normal method
    class Filter {
        function __construct() {}
     
        // Not a constructor
        function filter($a) {}
    }
     
    //But defining filter first raises an E_STRICT
    class Filter {
     
        // Not a constructor…
        function filter($a) {}
     
        // This raises E_STRICT
        function __construct() {}
    }
    
### Removal of date.timezone Warning
移除了当不设置date.tiemzone配置项时抛出的异常

### Removal of Alternative PHP Tags
移除`<% ( <%=), %>, <script language="php">, </script>`

### Removal of Multiple Default Blocks in Switch Statements
`switch-case-default`中default只有一个

### Removal of Redefinition of Parameters with Duplicate Names

    function foo($version, $version)
    {
        return $version;
    }
    
    echo foo(5, 7);
    
    // Pre PHP 7 result
    7
    
    // PHP 7+ result
    Fatal error: Redefinition of parameter $versionin /redefinition-of-parameters.php

### Removal of Dead Server APIs
以下SAPI内置函数去除
* sapi/aolserver
* sapi/apache
* sapi/apache_hooks
* sapi/apache2filter
* sapi/caudium
* sapi/continuity
* sapi/isapi
* sapi/milter
* sapi/nsapi
* sapi/phttpd
* sapi/pi3web
* sapi/roxen
* sapi/thttpd
* sapi/tux
* sapi/webjames
* ext/mssql
* ext/mysql
* ext/sybase_ct
* ext/ereg

### Removal of Hex Support in Numerical Strings
十六进制的字符串写法去除

    var_dump(is_numeric('0x123'));
    var_dump('0x123' == '291');
    echo '0x123' + '0x123';
    
    // Pre PHP 7 result
    bool(true)
    bool(true)
    582
    
    // PHP 7+ result
    bool(false)
    bool(false)
    0
    
    var_dump((int) '0x123'); // int(0)
    
    //instead use it below
    var_dump(filter_var('0x123', FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX)); // int(291)
    
### Removal of Deprecated Functionality
* 移除ext/mysql扩展
* 移除ext/ereg扩展
* Assigning new by reference
* Scoped calls of non-static methods from an incompatible $this context (such as Foo::bar() from outside a class, where bar() is not a static method)

### Reclassification and Removal of E_STRICT Notices
Removes the E_STRICT notice, changes it to an E_DEPRECATED if the functionality will be removed in future, changes it to an E_NOTICE, or promotes it to an E_WARNING.

### Deprecation of Salt Option for password_hash(

### Error on Invalid Octal Literals
错误的八进制常量会报错

    echo 0678; // Parse error:  Invalid numeric literal in...
    
### substr() Return Value Change
当substr截断的开始位置和字符串长度一样，以前是返回false,现在返回"",其他情况不变

    var_dump(substr('a', 1));
    
    // Pre PHP 7 result
    bool(false)
    
    // PHP 7+ result
    string(0) ""
