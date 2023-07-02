## Closure::call

(PHP 7, PHP 8)

Closure::call  —  Binds and calls the closure ,绑定并调用闭包

---------
### Description 说明

    public  Closure::call(object  $newThis,  mixed ...$args):  mixed

Temporarily binds the closure to  `newThis`, and calls it with any given parameters.
暂时将闭包绑定到 `newThis`，并使用任意给定的参数调用它。

---------
### Parameters 参数

`newThis`
The  object  to bind the closure to for the duration of the call.
在调用期间将闭包绑定到 object。

`args`
Zero or more parameters, which will be given as parameters to the closure.
零个或多个参数，他们将作为参数传递给闭包。

### Return Values 返回值

Returns the return value of the closure.
返回闭包的返回值。

----------
### Examples 示例

**Example #1  **Closure::call()**  example**

    <?php  
    class Value {  
	    protected $value;  
	    public function __construct($value) {  
		    $this->value = $value;  
	    }  
	      
	    public function getValue() {  
		    return $this->value;  
	    }  
    }  
      
    $three = new Value(3);  
    $four = new Value(4);  
      
    $closure = function ($delta) { var_dump($this->getValue() + $delta); };  
    $closure->call($three, 4);  
    $closure->call($four, 4);  
    ?>

The above example will output:

    int(7)
    int(8)

解释:用call方法,而不用bind或bindTo,因为call方法更直接方便.等于是绑定和调用一步完成.
我们用bind和bindTo来完成相同的功能

    //bind
    $closureBind = Closure::bind($closure, $three, 'Value');
    $closureBind(4);
    
    //bindTo
    $closureBindTo = $closure->bindTo($four, 'Value');
    $closureBindTo(4);
    结果:
    int(7)
	int(8)

但是我们要注意:如果闭包需要访问私有或受保护成员,那就不能使用call方法了.除非那个闭包是定义在类的内部.

----------
### User Contributed Notes 用户笔记

笔记1

**php-net at gander dot pl** 19-Aug-2021 10:23

You can also access private data:  
 你依然可以连接私有成员

    <?php  
    class Value {  
	    private $value;  
      
	    public function __construct($value) {  
		    $this->value = $value;  
	    }  
    }  
      
    $foo = new Value('Foo');  
    $bar = new Value('Bar');  
      
    $closure = function () { var_dump($this->value); };  
    $closure->call($foo);  
    $closure->call($bar);  
    ?>  

  
Output:  

    string(3) "Foo"  
    string(3) "Bar"

注意: 使用call方法,闭包会默认同时绑定$this和对应的类作用域.等同于bind和bindTo同时绑定对象和类作用域;

----------
笔记2

**sergey dot nevmerzhitsky at gmail dot com** 16-Sep-2016 10:12

Prior PHP 7.0 you can use this code:  
较早前的PHP 7.0你可以用下面的代码:
  

    <?php  
    class  A {
		public  $a;
	}
	$newthis = new  A();
    $cl = function($add) { return $this->a + $add; };  
      
    $cl->bindTo($newthis);  
    return call_user_func_array($cl, [10]);  
    ?>  

  
But this bind the closure permanently! Also read the article for Closure::bindTo() about binding closures from static context.
但这个绑定是永久的!
注意:这个例子在PHP8已经不行了.会报错: Using $this when not in object context ,因为`$cl`里面并不存在`$this`,只有在它的重新返回的那个闭包里面才有`$this`
需要改写成:

     $cl2 = $cl->bindTo($newthis);  
     return call_user_func_array($cl2, [10]);  结果:int(10)
