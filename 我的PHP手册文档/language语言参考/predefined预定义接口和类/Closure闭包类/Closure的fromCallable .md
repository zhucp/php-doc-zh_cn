## Closure::fromCallable

(PHP 7 >= 7.1.0)

Closure::fromCallable — Converts a callable into a closure ,将 callable 转换为闭包,把一个可调用的类型(可能是具名函数)转为一个闭包类型.(具名函数变成匿名函数)

-----------
### 说明

    public static Closure::fromCallable(callable $callback): Closure

Create and return a new anonymous function from given callback using the current scope. This method checks if the callback is callable in the current scope and throws a TypeError if it is not.
使用当前范围(作用域)从传过来的参数 `callback` 创建并返回一个新的 匿名函数。 此方法可以检查 `callback` 函数在当前作用域是否可调用， 如果不能，就抛出 TypeError。

> **Note**:
> 
> 从 PHP 8.1.0 开始，First-class 可调用语法 的语义与此方法相同。具体查看First-class 章节,这两个写法作用是一样的.

-----------
### 参数

`callback`

要转换的回调。这个回调就是一个匿名函数

### 返回值

返回新创建的 Closure， 或者如果 `callback` 在当前作用域无法调用， 则抛出 TypeError。

------------
### 参见

-   匿名函数
-   First-class 可调用语法
----------
### User Contributed Notes 用户笔记

笔记1

**nakerlund at gmail dot com** 23-May-2018 07:20

I have two points:  我想说两点:
  
It is possible to use Closure::fromCallable to convert private/protected methods to closures and use them outside the class.  
1:Closure::fromCallable转换后,可以令闭包在外部访问私有/受保护的成员
  
Closure::fromCallable accepts late dynamic bindings using the keyword static if provided as a string.  
2:Closure::fromCallable 使用字符串'static'可以接受'延迟静态绑定',能够帮助实现多态,区别于self关键字,self指的是定义时的类,static指的是调用时(运行时)的当前类。
 
My code below demonstrate how a private static method can be used as a callback in a function outside the class.  
我下面的代码,说明一个类的私有的静态的方法如何在类的外部,在另外一个方法中作为一个回调方法被调用 。
  

    <?php  ]
    //创建这个方法只是为了证明可以像上面说的那样做.功能上完全可以合并到mapUCFirst 里面去
    function myCustomMapper ( Callable $callable, string $str ): string {  
	    return join(' ', array_map( $callable, explode(' ', $str) ) );  
    }  
      
    class MyClass {  
      
	    public static function mapUCFirst ( string $str ): string {  
		    $privateMethod = 'static::mapper';//php8.0和8.1都还正常,但php8.2会报错: Deprecated: Use of "static" in callables is deprecated
			// $privateMethod = 'self::mapper'; //php8.2会报错:Deprecated: Use of "self" in callables is deprecated
			// $privateMethod = 'parent::mapper'; //php8.2会报错:Deprecated: Use of "parent" in callables is deprecated
			//总之php8.2已经不允许(不建议)你使用静态的callable作为fromCallable的参数
		    $mapper = Closure::fromCallable( $privateMethod );  
		    return myCustomMapper( $mapper, $str );  
	    }  
	    
	    private static function mapper ( string $str ): string {  
		    return ucfirst( $str );  
	    }  

    }  
      
    echo MyClass::mapUCFirst('four little uncapitalized words');  
    // Prints: Four Little Uncapitalized Words  正常结果:把每个单词的首字母变成大写
    总结:这个用户写的这个调用方式,只是证明了可以这样做,但实际开发中不会有人这么搞吧.
    ?>

----------

笔记2

**4-lom at live dot de** 09-May-2018 01:16

Sadly, your comparison is incorrect.  
遗憾的是，你的比较是不正确的 . (我的什么和什么比较?)
  
// The equivalent to   相当于

    $cl1 = Closure::fromCallable("getName");  
    $cl1 = $cl1->bindTo($bob, 'A');  

  
// is most likely this   很可能是这样

    $cl2 = function() {  
	    return call_user_func_array("getName", func_get_args());  
    };  
    $cl2 = $cl2->bindTo($bob, 'A');  

作者意思就是说上面这两种写法是相等的.
  
Executing one or the other Closure should result in the same access violation error you already postet.  
  执行一个或另一个闭包应该会导致与您已经发布的(??)相同的访问冲突错误。
  这个用户的笔记都不知道他在写些什么,无法理解他文字,只能看代码来理解了.

A simple PHP 7.0 polyfill could look like this:  一个简单的PHP7.0垫片(解决方案)可以像这样写

  

    namespace YourPackage;  
      
    /**  
    * Class Closure  
    *  
    * @see \Closure  
    */  
    class Closure  
    {  
	    /**  
	    * @see \Closure::fromCallable()  
	    * @param callable $callable  
	    * @return \Closure  
	    */  
	    public static function fromCallable(callable $callable)  
	    {  
		    // In case we've got it native, let's use that native one!  如果有原生的fromCallable,就使用原生的
		    if(method_exists(\Closure::class, 'fromCallable')) {  
			    return \Closure::fromCallable($callable);  
		    }  
      
		    return function () use ($callable) {  
			    return call_user_func_array($callable, func_get_args());  
		    };  
	    }  
    }

// 这个用户就是写了一个使php7.0也能兼容使用fromCallable这个方法的垫片.因为fromCallable是7.1.0才有的.

----------
笔记3
**igorchernin at yahoo dot com** 07-May-2017 08:13

It seems that the result of the "fromCallable" behaves a little bit different then an original Lambda function.  
 好像 fromCallable表现出来的结果与原来的闭包有点不同.
  

    class A {  
	    private $name;  
	    public function __construct($name)  
	    {  
		    $this->name = $name;  
	    }  
    }  
      
    // test callable  
    function getName()  
    {  
	    return $this->name;  
    }  
    $bob = new A("Bob");  
      
    $cl1 = Closure::fromCallable("getName"); //把getName方法转为一个匿名函数
    $cl1 = $cl1->bindTo($bob, 'A');  //给匿名函数绑定$this 和类作用域
    这一句:7.2就会报错Cannot rebind scope of closure created by ReflectionFunctionAbstract::getClosure()
    8.2 会报错:Cannot rebind scope of closure created from function
    也就是说不能给一个从方法转化而来的闭包绑定类作用域,顶多只能绑定$this.这样限制的原因,暂时还找到官方的解释文档
      
    //This will retrieve: Uncaught Error: Cannot access private property A::$name  
    //这会导致:不能访问私有变量$name
    $result = $cl1();  
    echo $result;  
      
    //But for a Lambda function   但是如果原生就是一个闭包
    $cl2 = function() {  
	    return $this->name;  
    };  
    $cl2 = $cl2->bindTo($bob, 'A');  
    $result = $cl2();  
      
    // This will print Bob  这会正常输出
    echo $result;

总结: 这个问题无法研究了,因为在新版一点的Php已经无法运行了.没有研究的必要了,可能Php官方已经修复了.
