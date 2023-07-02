##  First class callable syntax

### 说明
The first class callable syntax is introduced as of PHP 8.1.0, as a way of creating anonymous functions from callable. It supersedes existing callable syntax using strings and arrays. The advantage of this syntax is that it is accessible to static analysis, and uses the scope at the point where the callable is acquired.
PHP 8.1.0引入了首类可调用语法，作为从可调用类型转换为匿名函数的一种方式。它取代了使用字符串和数组的现有可调用语法。这种语法的优点是，静态分析可以访问它，并且为它绑定被定义时的那个地方的作用域和`$this` (不是被转换时的那个地方的作用域)。

`CallableExpr(...)` syntax is used to create a Closure object from callable. `CallableExpr` accepts any expression that can be directly called in the PHP grammar:
`CallableExpr(...)`语法结构是用于从callable类型转换创建为一个闭包对象的,它接收任何在PHP语法中能够直接调用的表达式:

----------
### 示例
**Example #1 Simple first class callable syntax**

    <?php  
    class Foo {  
	    private  $a = 1;
	    private  static  $s = 'I am static';
	    public function method() {}  
	    public static function staticmethod() {}  
	    public function __invoke() {}  
    }  
    $obj = new Foo();  
    $classStr = 'Foo';  
    $methodStr = 'method';  
    $staticmethodStr = 'staticmethod';  
    下面展示各种使用方式:
    $f1 = strlen(...);  //直接用函数名转化 ,等同于: Closure::fromCallable('strlen');
    
    $f2 = $obj(...); // invokable object  用调用函数的方式调用对象,就会触发__invoke魔术方法,所以要转化的函数就是__invoke
    var_dump($f2);
    打印的结果:
    object(Closure)#2 (2) {
	  ["function"]=>
	  string(13) "Foo::__invoke"
	  ["this"]=>
	  object(Foo)#1 (1) {
	    ["a":"Foo":private]=>
	    int(1)
	  }
	}
    
    $f3 = $obj->method(...);  
	var_dump($f3);
	结果:
		object(Closure)#2 (2) {
		  ["function"]=>
		  string(11) "Foo::method"
		  ["this"]=>
		  object(Foo)#1 (1) {
		    ["a":"Foo":private]=>
		    int(1)
		  }
		}
	//$f3 转化后的闭包,自动绑定了$this和类作用域,跟method被定义时的地方是一样的,也就是新的闭包与原生函数的作用域保持一致,此时闭包是可以访问私有变量$a的

    $f4 = $obj->$methodStr(...);   //$methodStr的值是method,所以等同于$obj->method(...)
	
    $f5 = Foo::staticmethod(...);  
    //$f5 = Closure::fromCallable([Foo::class, 'staticmethod']); //用fromCallable方法实现
	var_dump($f5);
	结果:
	object(Closure)#2 (1) {
	  ["function"]=>
	  string(17) "Foo::staticmethod"
	}
	//静态方法里面是没有$this的,但此时是可以访问静态私有变量$s,作用域依然保持一致.
 
    $f6 = $classStr::$staticmethodStr(...);   //等同于Foo::staticmethod(...)
    
    // traditional callable using string, array   用传统的方式:字符串,数组
    $f7 = 'strlen'(...);  //等同于 $f1 = strlen(...);
    $f8 = [$obj, 'method'](...);   //等同于  $f3 = $obj->method(...);  
    $f9 = [Foo::class, 'staticmethod'](...);  //等同于 $f5 = Foo::staticmethod(...); 也等同于 $f5 = Closure::fromCallable([Foo::class, 'staticmethod']);
    
    ?>
总结: 上述9种使用方式,归根结底就是:那个本体(原生函数)必须是callable 类型,你可以用各种表达方式指向(找到)那个callable .
另外就是新的闭包的类作用域是保持与原生函数一致的.原生函数本来有`$this` 和类作用域的, 转化出来的闭包也有相同的.原生函数本来就没有`$this`和类作用域的,转化出来后也是没有的.并且php7开始,已经不允许给这个闭包绑定新的类作用域了,会报错: Cannot rebind scope of closure created from function 不能给一个从方法转化而来的闭包绑定类作用域

----------
 **Note**: 注意
 The `...` is part of the syntax, and not an omission. 3个点语法是语法的一部分,不是表示省略号
 
----------

`CallableExpr(...)` has the same semantics as `Closure::fromCallable()`. That is, unlike callable using strings and arrays, `CallableExpr(...)` respects the scope at the point where it is created:
`CallableExpr(...)`拥有和`Closure::fromCallable()`一样的语义,作用是一样的.还有就是,不像使用字符串和数组的callable , `CallableExpr(...)`得到的闭包依然遵循(绑定)创建它(原生方法)的那个位置的作用域:

----------
`CallableExpr(...)` 与传统 callable 的类作用域比较
**Example #2 Scope comparison of `CallableExpr(...)` and traditional callable**

    <?php  
    class Foo {  
    public function getPrivateMethod() {  
    return [$this, 'privateMethod'];  
    }  
    private function privateMethod() {  
    echo __METHOD__, "\n";  
    }  
    }  
    $foo = new Foo;  
    $privateMethod = $foo->getPrivateMethod();  
    $privateMethod();  
    // Fatal error: Call to private method Foo::privateMethod() from global scope  
    // This is because call is performed outside from Foo and visibility will be checked from this 
    point.  
	/*
		var_dump($privateMethod);
		array(2) {
		[0]=>
		object(Foo)#1 (0) {
		}
		[1]=>
		string(13) "privateMethod"
		}
		$privateMethod 是一个数组,并不是闭包!这个数组指明了要调用哪个方法,等同于$foo->privateMethod(); 在类的外部调用调用私有方法肯定是不行的
		*/
	
    class Foo1 {  
    public function getPrivateMethod() {  
    // Uses the scope where the callable is acquired.  使用callable 定义时所在地的作用域
    return $this->privateMethod(...); // identical (完全相同) to Closure::fromCallable([$this, 'privateMethod']);  
    }  
    private function privateMethod() {  
    echo __METHOD__, "\n";  
    }  
    }  
    $foo1 = new Foo1;  
    $privateMethod = $foo1->getPrivateMethod();  
    $privateMethod(); // Foo1::privateMethod  
    var_dump($privateMethod);
    /*
    object(Closure)#3 (2) {
	  ["function"]=>
	  string(19) "Foo1::privateMethod"
	  ["this"]=>
	  object(Foo1)#2 (0) {
	  }
	}
	此时$privateMethod是一个闭包,并且$this指向$foo1,类作用域是Foo1 ,因为在闭包绑定了原生函数(callable )定义时所在的地方的作用域
	*/
    ?>


----------

 **Note**:注意点
 
 Object creation by this syntax (e.g `new Foo(...)`) is not supported, because `new Foo()` syntax is not considered a call.
 对象创建由这种语法(例如`new Foo(...)`)是不被支持的,因为`new Foo()`语法不被认为是一种调用. 这是生成一个新的对象 .如果你对象后面加括号,就可以认为是一种调用,因为这样会调用对象的__invoke魔术方法.
 
 **Note**:注意点
 
The first-class callable syntax cannot be combined with the nullsafe operator. Both of the following result in a compile-time error:
首类可调用语法无法与nullsafe(空白安全)运算符结合使用.下面这两句都会导致编译时错误:
关于空白安全运算符(?->),可以了解PHP8新特性: nullsafe ,简单来说:当运行结果不是Null,才会继续执行箭头后面的语句
 <?php  
 $obj?->method(...);   
 $obj?->prop->method(...);  
 //这两句都会导致: PHP Fatal error:  Cannot combine nullsafe operator with Closure creation
 ?>

### User Contributed Notes 用户笔记
笔记1
bienvenunet at yahoo dot com
**5 days ago**

There's a major gotcha with this syntax that may not be apparent until you use this syntax and find you're getting "Cannot rebind scope of closure created from method" exceptions in some random library code.  
在使用此语法并发现在一些随机库代码中出现“无法重新绑定从方法创建的闭包的范围”异常之前，此语法的一个主要问题可能并不明显

As the documentation indicates, the first-class callable uses the scope at the point where the callable is acquired. This is fine as long as nothing in your code will attempt to bind the callable with the \Closure::bindTo method.  
正如文档所言,首类可调用使用的作用域是  callable 被定义的地方的作用域.这个会正常运行只要没有代码试图去绑定callable 使用\Closure::bindTo方法.

I found this the hard way by changing callables going to Laravel's Macroable functionality from the array style to the first-class callable style. The Macroable functionality \Closure::bindTo calls on the callable.  
 我发现在Laravel's Macroable 中很难从数组方式变成首类可调用方式.因为 Macroable都是使用 \Closure::bindTo来调用callable的
  
AFAIK(As far as I know), the only workaround is to use the uglier array syntax.
就我而言,唯一绕过这个问题的方式是继续使用丑陋的数组语法.

这个用户提出的一个使用上的麻烦之处,就是他用的Laravel框架都是用bindTo方式来绑定闭包,但这种方式一旦使用的闭包是从首类可调用语法转化过来的闭包,就会报错:Cannot rebind scope of closure created from method;
这种问题只能等Laravel框架升级处理.在此之前还是使用传统的数组方式.首类可调用语法属于PHP8的新特性,只能等各类框架自行适应.
