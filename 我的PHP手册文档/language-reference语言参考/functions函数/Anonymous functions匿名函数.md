## 匿名函数

Anonymous functions, also known as `closures`, allow the creation of functions which have no specified name. They are most useful as the value of [callable](https://www.php.net/manual/en/language.types.callable.php) parameters, but they have many other uses.
匿名函数（Anonymous functions），也叫闭包函数（closures），还可以叫做 lamda表达式(在java,c#,c++,python中),允许临时创建一个没有指定名称的函数。最经常用作回调函数 callable类型参数的值。当然，也有其它应用的情况。

匿名函数目前是通过 Closure 类来实现的。也就是说匿名函数这种语法，实际上是PHP在内部把这种表达式转换成内置类Closure类的对象实例。

----------
**Example #1 匿名函数示例**

    `<?php  
    //执行一个正则表达式搜索并且使用一个回调函数进行替换，这个回调函数就是一个匿名函数
    echo preg_replace_callback('~-([a-z])~', 
    function ($match) {  
    	return strtoupper($match[1]);  
    }, 
    'hello-world');  
    // 输出 helloWorld  
    ?>`

这个例子展示了用匿名函数的第一个优点：这个匿名函数只会在这一个地方使用到，仅适用于这个地方，其他地方根本不会用到这个函数，我们也就没有必要专门为这个函数去创建一个独立的function，从而避免了‘污染’，因为多一个函数就多了一个节点.而且最重要,我们不必去为这个函数叫什么名字而苦恼.
第二个优点是,这样可以让你的代码在结构上放在一起,一看就能看完全部代码内容.而不需要跳去独立function那边去看代码.

----------
Closures can also be used as the values of variables; PHP automatically converts such expressions into instances of the [Closure](https://www.php.net/manual/en/class.closure.php) internal class. Assigning a closure to a variable uses the same syntax as any other assignment, including the trailing semicolon:
闭包函数也可以作为变量的值来使用。PHP 会自动把此种表达式转换成内置类 Closure 的对象实例。把一个 closure 对象赋值给一个变量的方式与普通变量赋值的语法是一样的，最后也要加上分号(这是一段变量赋值语法!不是函数创建语法,记得最后加';',不然就是语法错误)：

**Example #2 匿名函数变量赋值示例**

    `<?php  
	    $greet = function($name) {  
		    printf("Hello %s\r\n", $name);  
	    };  
	    $greet('World');  
	    $greet('PHP');  
    ?>`
当你`var_dump($greet);`的时候,结果是:`object(Closure)#1 (0) {}`,这里可以表明匿名函数本质是一个Closure类.

注意:我们知道一个类的一般的使用流程是这样的:1声明(declare),叫什么(名字),有什么(属性),能干什么(方法),2赋值(assign,也叫分配,因为在分配内存嘛)到某个变量,这个动作也叫实例化.3调用,`$a->doSomething()`;
 那么我们理清一下匿名函数整个使用流程: 匿名函数本质是一个类(闭包类),它也有声明,就是那段语法: `function(){}` ,然后赋值`$a = function(){};` ,最后是调用:$a(); 很多时候,我们会把声明和赋值写在一起,看起来是一步完成的.这个就是匿名形式的写法特点.因为在PHP中匿名函数的本质不是函数,也是类.所以匿名类也是这样的使用流程,



----------
Closures may also inherit variables from the parent scope. Any such variables must be passed to the `use` language construct. As of PHP 7.1, these variables must not include [superglobals](https://www.php.net/manual/en/language.variables.predefined.php), $this, or variables with the same name as a parameter. A return type declaration of the function has to be placed _after_ the `use` clause.
闭包可以从父作用域(外部)中继承变量。 任何此类变量都应该用 `use()` 语言结构传递进去。 PHP 7.1 起，不能传入此类变量： superglobals、 $this 或者和参数重名。 匿名函数的返回类型声明必须放在 `use()` 子句的后面 。

闭包有自身作用域(闭包函数内部)和父作用域(定义这个函数的地方,闭包函数外部),作用域之间是隔离的.通过使用use从而使匿名函数在内部可以使用外部的变量。 虽然参数也能传递到内部,但use传递让你不需要每次调用都写明要传递的变量.

**Example #3 从父作用域继承变量**

    `<?php  
    $message = 'hello';  
      
    // 没有 "use"  
    $example = function () {  
	    var_dump($message);  
    };  
    $example();   //会报warning:Undefined variable $message,但会输出结果:NULL
      
    // 继承 $message  
    $example = function () use ($message) {  
	    var_dump($message);  
    };  
    $example();   //结果输出:string(5) "hello"
      
    // 当函数被定义而不是被调用的时候继承变量的值 ,继承的变量的值取决于匿名函数被定义时的值,而不是调用时的值,因为定义时,闭包类会把这些use传过来的变量复制到内部静态变量数组中,已经与外部的原始变量没有关系了
    $message = 'world';  
    $example();  /这里的结果是输出函数定义时的值:string(5) "hello",而不是后面变化后的world
      
    // 重置 message  
    $message = 'hello';  
      
    // 通过引用继承(引用传递)  
    $example = function () use (&$message) {  
	    var_dump($message);  
    };  
    $example();  //用&符号进行引用传递,就变成始终取决于调用时的变量的值,结果:string(5) "hello",因为上面重置了值
      
    // 父级作用域改变的值反映在函数调用中  
    $message = 'world';  
    $example();  //结果:string(5) "world" ,因为上一行刚改变了值
      
    // 闭包函数也可以接受常规参数,注意写法:普通参数在前,use在后
    $example = function ($arg) use ($message) {  
	    var_dump($arg . ' ' . $message);  
    };  
    $example("hello");  //结果:string(11) "hello world"
      
    // 正确写法:函数的返回类型在 use 子句的后面  
    $example = function () use ($message): string {  
	    return "hello $message";  
    };  
    var_dump($example());  //结果:string(11) "hello world"
    ?>`

以上示例的输出类似于：

    Notice: Undefined variable: message in /example.php on line 6
    NULL
    string(5) "hello"
    string(5) "hello"
    string(5) "hello"
    string(5) "world"
    string(11) "hello world"
    string(11) "hello world"

As of PHP 8.0.0, the list of scope-inherited variables may include a trailing comma, which will be ignored.
从 PHP 8.0.0 开始，作用域继承的变量列表可能包含一个尾部的逗号，这个逗号将被忽略。

这里补充一下:如果使用引用传递,变量的定义甚至可以在函数定义之后.也就是变量还不存在,就可以先use它.只不过在匿名函数内部这个值是NULL,但不会报'变量不存在'的警告.当然这种写法不推荐! 当后来又定义了同名变量,再次调用匿名函数,此时会指向这个后来新创建的同名变量.

    <?php
    //如果use 引用的变量还不存在,后面才声明的.那么还是会指向同一个值吗?答案是:会的
    $afunc = function()use(&$v){
        var_dump($v);
    };
    
    $afunc(); //不会报错,输出NULL
    
    $v =1;
    
    $afunc(); //输出 int(1)


Inheriting variables from the parent scope is _not_ the same as using global variables. Global variables exist in the global scope, which is the same no matter what function is executing. The parent scope of a closure is the function in which the closure was declared (not necessarily the function it was called from). See the following example:
*这些变量都必须在函数或类的头部声明*(这句话,官方中文文档有,但英文文档没有找到对应的语句,可能意思是指:要use继承的变量必须在头部先声明好,不能像我们上面那个例子那样,先use后声明)。 
从父作用域中继承变量与使用全局变量是不同的。全局变量存在于一个全局的范围，无论当前在执行的是哪个函数。而 闭包的父作用域是定义该闭包的函数(或者说是那片代码块)（不一定是调用它的函数）。示例如下：

**Example #4 Closures 和作用域**

    `<?php  
    // 一个基本的购物车，包括一些已经添加的商品和每种商品的数量。  
    // 其中有一个方法用来计算购物车中所有商品的总价格，该方法使  
    // 用了一个 closure 作为回调函数。  
    class Cart  
    {  
	    const PRICE_BUTTER = 1.00;  
	    const PRICE_MILK = 3.00;  
	    const PRICE_EGGS = 6.95;  
	      
	    protected $products = array();  
	      
	    public function add($product, $quantity)  
	    {  
		    $this->products[$product] = $quantity;  
	    }  
	      
	    public function getQuantity($product)  
	    {  
		    return isset($this->products[$product]) ? $this->products[$product] : FALSE;  
	    }  
	      
	    public function getTotal($tax)  
	    {  
		    $total = 0.00;   
		    $callback = function ($quantity, $product) use ($tax, &$total)  //对总价使用引用传递,所以匿名函数内容对于总价的值改变会作用于外部的总价
			    {  //获取商品单价,这里 __CLASS__获取当前的类名,然后组合出顶部常量的名称,再用constant方法动态获取常量值
				    $pricePerItem = constant(__CLASS__ . "::PRICE_" .  
				    strtoupper($product));  
				    $total += ($pricePerItem * $quantity) * ($tax + 1.0);  
			    };  
		    array_walk($this->products, $callback);  //对每一个数组元素(产品)都执行一次匿名函数
		    return round($total, 2);  
		}  
    }  
      
    $my_cart = new Cart;  
      
    // 往购物车里添加条目  
    $my_cart->add('butter', 1);  
    $my_cart->add('milk', 3);  
    $my_cart->add('eggs', 6);  
      
    // 打出出总价格，其中有 5% 的销售税.  
    print $my_cart->getTotal(0.05) . "\n";  
    // 最后结果是 54.29  
    ?>`

这里例子四也完全可以不使用匿名函数来实现,这里只是为了展示匿名函数的使用方法.这种方式也更好理解,更好看.

----------
**Example #5 自动绑定 `$this`**
当在类中声明匿名函数时，当前的类会自动与之绑定，使得 $this 在匿名函数的中可用。

`<?php  
  
	class Test  
	{  
		public function testing()  
		{  
			return function() {  
				var_dump($this);  
			};  
		}  
	}  
	  
	$object = new Test;  
	$function = $object->testing();  
	$function();  
  
?>`

以上示例会输出：

    object(Test)#1 (0) {
    }

When declared in the context of a class, the current class is automatically bound to it, making `$this` available inside of the function's scope. If this automatic binding of the current class is not wanted, then [static anonymous functions](https://www.php.net/manual/en/functions.anonymous.php#functions.anonymous-functions.static) may be used instead.
当在类的上下文中(在类中)声明闭包时，当前的类会自动与之(闭包)绑定，使得 `$this` 在匿名函数的作用域(内部)中可用。如果不需要当前类的自动绑定，可以使用 静态匿名函数 替代(因为静态函数中不允许存在实例($this))。

补充一下:子类继承父类,在父类中`$this`指向父类的实例,在子类中`$this`指向子类的实例.记住`$this`只会指向实例化时的类的实例

### 静态匿名函数

Anonymous functions may be declared statically. This prevents them from having the current class automatically bound to them. Objects may also not be bound to them at runtime.
匿名函数允许被定义为静态化。这样可以防止当前类自动绑定到它们身上，对象在运行时也不能被绑定到它们上面(看下面的例子7)。

**Example #6 试图在静态匿名函数中使用 `$this`**

    <?php  
      
    class Foo  
    {  
	    function __construct()  
	    {  
		    $func = static function() {  
		    var_dump($this);  
	    };  
	    $func();  
	    }  
	    };  
	    new Foo();  
      
    ?>

以上示例会输出：

    Notice: Undefined variable: this in %s on line %d 
    //在php 5.4以上,会报错:PHP Fatal error:  Uncaught Error: Using $this when not in object context 
    NULL

**Example #7 试图将对象绑定到静态匿名函数**

    `<?php  
      
    $func = static function() {  
	    // function body  
    };  
    $func = $func->bindTo(new stdClass);  
    $func();  
      
    ?>`

以上示例会输出：

    Warning: Cannot bind an instance to a static closure in %s on line %d //不能绑定一个实例到静态闭包函数中
    //Php5.4以后: Warning: Cannot bind an instance to a static closure

具体理解需要去了解 :  Closure::bind()  ,在静态匿名函数中,你不能绑一个对象实例进去,因为静态函数里面不允许存在$this.

----------
### 更新日志

版本 :7.1.0

说明:匿名函数不能用 use 包含 superglobals 、$this 、 跟参数同名的变量。

### 注释

> **Note**: 可以在闭包中使用 func_num_args()，func_get_arg() 和 func_get_args()。
----------
### User Contributed Notes  用户贡献的笔记
笔记1
**jake dot tunaley at berkeleyit dot com** 07-Jan-2019 05:04

Beware of using `$this` in anonymous functions assigned to a static variable.  
把匿名函数赋值给静态变量的时候,注意里面的`$this`。
在一个类中,把一个匿名函数分配(赋值)给一个静态变量,此时要小心注意$this所指向的对象是谁!  原因在于静态变量只会在第一次声明时初始赋值一次,后面再声明也不会进行初始赋值.而是直接从内存中获取,不会重新创建,自然也不会重新初始赋值!还有就是:类的方法内部定义的静态变量,在类的所有实例中的这个方法内都是共享的!就是说:同一个方法,共享同一个内部静态变量.不同的方法,不能使用其他方法内定义的变量.只能共享头部定义的变量.

    <?php  
        class Foo {  
            public function bar() {  
        	    static $anonymous = null;  
        	    if ($anonymous === null) {  
        		    // Expression is not allowed as static initializer workaround  静态变量是不允许使用表达式来初始赋值的!这里是一个变通方法.
        		    $anonymous = function () {   return $this;  };  
        	    }  
        	    return $anonymous();  
            }  
        }  
              
            $a = new Foo();  
            $b = new Foo();  
            var_dump($a->bar() === $a); // True  
            var_dump($b->bar() === $a); // Also true  
    ?>


In a static anonymous function, `$this` will be the value of whatever object instance that method was called on first.  
在一个静态匿名函数内部中的`$this`会指向那个第一次调用它的实例.后续其他相同类的实例,这个静态匿名函数里面的`$this`,也是那个第一次调用它的实例.导致this不是指向自己,这里不注意的话会出大问题.  
To get the behaviour you're probably expecting, you need to pass the `$this` context into the function.  
为了得到你期望的行为结果,你需要手动传递上下文中的(当前类中的)`$this`到匿名函数中.例如下面这个改写例子:
	      
    <?php  
    	    class Foo {  
	    	    public function bar() {  
		    	    static $anonymous = null;  
		    	    if ($anonymous === null) {  
			    	    // Expression is not allowed as static initializer workaround  
			    	    $anonymous = function (self $thisObj) {  //这里指明参数类型必须是self,代表自身类型,也就是class Foo 类型,这是严格写法,不写参数类型也是可以正常运行的
				    	    return $thisObj;  
			    	    };  
		    	    }  
		    	    return $anonymous($this); //这里的$this,在上下文中,指的是当前实例!在这里指明传入指向当前实例的$this,就避免了上面的例子的问题.  
	    	    }  
    	    }  
    	      
    	    $a = new Foo();  
    	    $b = new Foo();  
    	    var_dump($a->bar() === $a); // True  
    	    var_dump($b->bar() === $a); // False  
        ?>

解释:php是自动传递外面的`$this`到匿名函数内部的, 不需要使用use().但是如果这个匿名函数赋值给了一个静态变量,就会导致第一次赋值之后,里面的`$this`就固化了,永远指向第一次声明时的那个实例`$this`.而其他实例在调用时,都是共用同一个静态变量,静态变量的内容也没有变化(因为有判断`$anonymous === null`,导致后面不会重复执行匿名函数赋值给`$anonymous`的操作).就会导致在其他实例中`$this`竟然不是指向自己,而是指向第一次的那个实例. 优化代码之后,匿名函数每次调用都手动传递当前实例`$this`作为参数给匿名函数(参数命名不能叫`$this`,这个是php规定的,因为已经存在默认的`$this`,随便叫个`$thisObj`表示当前object的`$this`),这样就解决了`$this`不是指向自己的问题.

如果去除`$anonymous === null`的判断,每次调用bar()函数,都会进行一次`$anonymous`的二次赋值.那么也能解决掉这个`$this`不指向'自己'的问题.但是这样做的话,设成静态变量就好像没有意义了.每次都重新赋值,不就等于是普通变量了吗?
在实际开发中,这种情况很少会遇到,即使遇到了,也应该从设计上解决,而不是用这样的方法去打补丁!这种方式不好看,不优雅(在匿名函数内部同时存在`$this` 和 `$thisObj`),也不好理解.

下面我们继续改写这个例子,来理解静态变量只会初始赋值一次,和同时存在`$this` 和`$thisObj`

    public  function  bar() {
	    static  $anonymous = null;
	    if(isset($anonymous)) var_dump($anonymous);
	    if ($anonymous === null) {
		    $anonymous = function (self  $thisObj) { 
			    var_dump($this);
			    var_dump($thisObj);
			    return  $thisObj;
		    };
	    }
	    return  $anonymous($this); 
    }

结果:
	
    //第一次调用,$anonymous是null,所以没有打印出$anonymous
    //打印出$this
    object(Foo)#1 (0) {
    }
     //打印出$thisObj
    object(Foo)#1 (0) {
    }
    
    bool(true) //两个的值是全等的
    
	//第二次调用,$anonymous是静态变量,保存着之前的值,直接从内存中获取,可以看到里面的this指向的Foo#1
    object(Closure)#3 (2) {
      ["this"]=>
      object(Foo)#1 (0) {
      }
      ["parameter"]=>
      array(1) {
        ["$thisObj"]=>
        string(10) "<required>"
      }
    }
	//打印出的this还是Foo#1
    object(Foo)#1 (0) {
    }
    //打印出的thisObj是Foo#2
    object(Foo)#2 (0) {
    }
    
    bool(true)

----------
笔记2

**dexen dot devries at gmail dot com** 07-Jun-2018 09:54

`Every instance of a lambda has own instance of static variables. This provides for great event handlers, accumulators, etc., etc.  
 每个匿名函数的实例都有自己的静态变量实例.这为事件句柄回调函数和计数器等等提供了可行性.(提供了独立的存储数据的能力)

Creating new lambda with function() { ... }; expression creates new instance of its static variables. Assigning a lambda to a variable does not create a new instance. A lambda is object of class Closure, and assigning lambdas to variables has the same semantics as assigning object instance to variables.  
创建一个新的闭包,它拥有自己的静态变量.一个匿名函数赋值给一个变量, ~~此时并没有创建一个新的实例~~ .匿名函数实质是一个闭包类, 将lambdas赋值给变量与将对象实例赋值给变量具有相同的语义(实质都是把一个类的实例赋值给一个变量)
解释:这段话,不清楚这个用户是什么用意,整段话基本没给干货,只有最后一句'相同语义'是比较有用, 这跟我在示例2后面的注意点讲的是同一件事,也就是匿名函数和普通的类具有相同的使用流程.所以说是语义是相同的. 
	但他这里还说匿名函数赋值给变量时,没有创建新的实例,这一句是错误的,看下面他给的例子,他自己都注释说了creates new instance .也就是在把一个匿名函数赋值给一个变量时, 是生成了一个新的实例的.
	如果真的要说哪里没有创建新的实例,那就是generate_lambda这个方法里面,的确没有创建新的实例,只是一个'声明',这里就是说如果想把匿名函数的'声明'和赋值(实例化)分开写,就把匿名函数的'声明'放到另外一个普通函数里面作为返回值,让普通函数返回一个匿名函数,然后在其他地方调用普通函数,结果赋值给一个变量.此时才生成了新的实例. 这种写法适用于需要多次创建多个功能相同的匿名函数,但又要让它们互相独立运行,不会相互影响静态成员.

Example script: $a and $b have separate instances of static variables, thus produce different output. However $b and $c share their instance of static variables - because $c is refers to the same object of class Closure as $b - thus produce the same output.  
示例脚本:a和b拥有隔离的实例的静态变量,因此输出不同结果.然后b和c共享它们的实例的静态变量,因为c是引用与b相同的闭包类的实例,因此输出相同结果.

  

      <?php  
              
            function generate_lambda() : Closure  
            {  
	            //creates new instance of lambda  说法错误:这里并没有创建新的实例,没有分配内存!!!仅仅是'声明',只有在真正调用,执行到这里的时候才会创建一个实例,然后返回.
	            return function($v = null) {  
		            static $stored;  
		            if ($v !== null)  
			            $stored = $v;  
		            return $stored;  
	            };  
            }  
              
            $a = generate_lambda(); # creates new instance of statics  这里赋值给变量,才是真正的创建实例的时候
            $b = generate_lambda(); # creates new instance of statics  
            $c = $b; # uses the same instance of statics as $b  
              
            $a('test AAA');  
            $b('test BBB');  
            $c('test CCC'); # this overwrites content held by $b, because it refers to the same object  
             //$c只是直接复制$b,在Php中对象实例的直接复制都是引用传递,实际它们两指向的内存都是同一块,如果想真正的复制需要用对象复制 clone 语法
            var_dump([ $a(), $b(), $c() ]); 
        ?>
        
    This test script outputs:  
        array(3) {  
        [0]=>  
        string(8) "test AAA"  
        [1]=>  
        string(8) "test CCC"  
        [2]=>  
        string(8) "test CCC"  
        }


我额外增加一个clone对象复制的例子给大家了解一下

    $d = clone  $b; //真正的对象完全复制,真的使用了另外一块内存
    $b('test BBB');
    $d('test DDD');
    var_dump([ $a(), $b(), $c(),$d() ]);

输出:

    array(4) {
      [0]=>
      string(8) "test AAA"
      [1]=>
      string(8) "test BBB" //b改回了bbb
      [2]=>
      string(8) "test BBB" //c引用b,所以结果和b一样
      [3]=>
      string(8) "test DDD" //d是真正的复制
    }


解释:`$a`和`$b`实质是两个内容一模一样的Closure类,但它们是各自独立的,没有联系的.就像两个普通类,内部一模一样,但类名不一样.那它们就是完全不相干的两个类,那么实例化后的内部同名静态变量当然也不会指向相同的内存地址.大家都无名,不能认为大家名字都一样,而是应该认为大家名字都不一样.
这里其实可以认为每次generate_lambda(),都完整地重写了一遍匿名函数的创建流程.这样两个闭包的'源头'都是独立的,又怎么会有关联呢.

----------
笔记3

**toonitw at gmail dot com** 27-Dec-2017 05:40

`As of PHP 7.0, you can use IIFE(Immediately-invoked function expression) by wrapping your anonymous function with ().  
Immediately-invoked function expression: 立即执行函数表达式(这个在js中有,但并没有找到php官方说明这个概念),用括号()来包装(包裹)匿名函数(一段表达式),使之可以立即执行,这样写的好处是代码更加简洁了!


    <?php  
	    $type = 'number';  
	    var_dump( ...( function() use ($type) {  
						    if ($type=='number') return [1,2,3];  
						    else if ($type=='alphabet') return ['a','b','c'];  
					    } 
					 )()
		);  
    ?>`

这段纠结在一起的代码,咋眼一看完全不理解.
解释:把匿名函数用括号包起来,使这段代码可执行.再在后面加一对括号表示立即调用这段可执行的代码.语法结构上就是连续两对小括号(这个样子有点奇特).匿名函数返回一个数组,3个点语法的作用是把这个数组分解成一个个参数,最后var_dump打印出来.
3个点语法:如果...后面是数组,那么就把这个数组按顺序分解成一个个参数,对应函数要求的参数.具体查看 [官方手册](https://www.php.net/manual/zh/functions.arguments.php) 的第11个例子开始.

总结上面例子: 这样的写法,连那个接收匿名函数的变量都省了,之前还需要一个变量`$a = function(){}` ,然后再`$a()`进行调用. 现在这种写法,等于是用括号,代替了`$a`.也就是 `$res = (function(){})()` 等价于 `$a = function(){};$res = $a();` 一句等于两句,还省下不用为创建`$a`而去想a的名字.

----------

笔记4

**ayon at hyurl dot com** 29-Apr-2017 05:35

`One way to call a anonymous function recursively is to use the USE keyword and pass a reference to the function itself:  
  递归调用匿名函数的一种方法是使用use关键字并传递对函数本身的引用


    <?php  
	    $count = 1;  
	    $add = function($count) use (&$add){  
		    var_dump($add);
		    $count += 1;  
		    if($count < 10) $count = $add($count); //recursive calling  递归调用
		    return $count;  
	    };  
	    echo $add($count); //Will output 10 as expected  
    ?>`

结果:


    object(Closure)#1 (2) {
      ["static"]=>
      array(1) {
        ["add"]=>
        *RECURSION*
      }
      ["parameter"]=>
      array(1) {
        ["$count"]=>
        string(10) "<required>"
      }
    }
    
    ......中间循环了9次,最后输出  
    10

解释:匿名函数中use 的东西,会被作为Closure的静态变量,如果我们加多一个`$a=2`到`use(&$add, $a)`,打印出来就是:

    ["static"]=>
      array(2) {
        ["add"]=>
        *RECURSION*
        ["a"]=>
        int(2)
      }

如果我们不使用引用,直接`use($add)`,就会报错:`Undefined variable $add` ,因为在创建这个匿名函数之前,并不存在这个`$add`变量;
但是如果我们使用引用,那么如果不存在`$add`变量的话,那么这个值就是NULL,也就引用到了一个不存在的值,但不会报错;
所以这个例子的代码执行流程就是:先创建了一个`$add`匿名函数,它创建时use只是指向`$add`,这个`$add`虽然还不存在,但php不会报错,php不认为这是一个致命问题,从逻辑上讲一个变量的值是NULL,是完全符合规定的.
当函数内部执行 `$count = $add($count)`时,此时Php才会去判断这个`$add`是什么东西,发现是一个匿名函数,然后就调用.
匿名函数使用use,可以很方便地把外部变量传到内部.如果不使用use,这个例子也可以递归,但是需要使用参数的形式来传递自己:

    <?php
    $count = 1;
    $add = function ($count, &$add) {
        $count += 1;
        if($count < 10) {
            $count = $add($count, $add);
        } //recursive calling 递归调用
        return $count;
    };
    echo $add($count, $add); //Will output 10 as expected

明显用use的方式会更好看和便捷.

----------

笔记5

**john at binkmail dot com** 07-Feb-2017 07:36


`PERFORMANCE BENCHMARK 2017!  
  2017年测试基准！
  
I decided to compare a single, saved closure against constantly creating the same anonymous closure on every loop iteration. And I tried 10 million loop iterations, in PHP 7.0.14 from Dec 2016. Result:  
 我决定将单个保存的闭包与在每次循环迭代中不断创建相同的匿名闭包进行比较。从2016年12月开始，我在PHP 7.0.14中尝试了1000万次循环迭代。结果：
 
a single saved closure kept in a variable and re-used (10000000 iterations): 1.3874590396881 seconds  
 保存在变量中并重复使用的单个已保存闭包（10000000次迭代）：1.387459039681秒
 
new anonymous closure created each time (10000000 iterations): 2.8460240364075 seconds  
 每次创建新的匿名闭包（10000000次迭代）：2.8460240364075秒
 
In other words, over the course of 10 million iterations, creating the closure again during every iteration only added a total of "1.459 seconds" to the runtime. So that means that every creation of a new anonymous closure takes about 146 nanoseconds on my 7 years old dual-core laptop. I guess PHP keeps a cached "template" for the anonymous function and therefore doesn't need much time to create a new instance of the closure!  
 换句话说，在1000万次迭代的过程中，在每次迭代中再次创建闭包只会给运行时增加总共“1.459秒”。这意味着，在我7岁的双核笔记本电脑上，每创建一个新的匿名闭包大约需要146纳秒。我猜PHP为匿名函数保留了一个缓存的“模板”，因此不需要太多时间来创建一个新的闭包实例！
  
So you do NOT have to worry about constantly re-creating your anonymous closures over and over again in tight loops! At least not as of PHP 7! There is absolutely NO need to save an instance in a variable and re-use it. And not being restricted by that is a great thing, because it means you can feel free to use anonymous functions exactly where they matter, as opposed to defining them somewhere else in the code. :-)

因此，您不必担心在紧密的循环中不断地重新创建匿名闭包！至少不是从PHP 7开始！绝对没有必要将实例保存在变量中并重复使用。不受此限制是一件很棒的事情，因为这意味着你可以自由地在重要的地方使用匿名函数，而不是在代码中的其他地方定义它们。：-）

一句话总结:不必因为效率问题,而用一个变量去保存一个匿名函数,然后再反复通过变量来调用.因为新创建一个闭包和通过变量来调用闭包,它们之间的效率差异非常非常非常小!!!小到1000万次循环也只是相差1.459秒

----------
笔记6

利用匿名函数给类动态新增方法

**erolmon dot kskn at gmail dot com** 19-Jun-2015 09:48

    <?php  
    /*  
    (string) $name Name of the function that you will add to class.  
    Usage : $Foo->add(function(){},$name);  
    This will add a public function in Foo Class.  因为php规定所有的重载(overloading)方法/属性都必须被声明为 public
    */  
    class Foo  
    {  
	    public function add($func,$name)  
	    {  
		    $this->{$name} = $func;  //给当前类添加一个新的属性,而属性的值是一个匿名函数
	    }  
	    public function __call($func,$arguments){  
		    call_user_func_array($this->{$func}, $arguments); 
		    //这里比较巧妙的处理:当调用的方法不存在时,类就会调用__call方法(该方法在调用的方法不存在时会自动调用),然后__call方法把'不存在'的方法名和参数传递给call_user_func_array,从而call_user_func_array根据属性名(方法名)获取了动态属性的值,而这个值就是一个匿名函数.这样正好就能够执行(callable)了.
	    }  
    }  
    $Foo = new Foo();  
    $Foo->add(function(){  
		    echo "Hello World";  
	    },
	    "helloWorldFunction");  
    $Foo->add(function($parameterone){  
		    echo $parameterone;  
	    },
	    "exampleFunction");  
	    
    $Foo->helloWorldFunction(); /*Output : Hello World*/  
    
    $Foo->exampleFunction("Hello PHP"); /*Output : Hello PHP*/  
    ?>

首先给类动态新增方法或属性,在PHP8.2开始被弃用了.会报错:Creation of dynamic property Foo::$helloWorldFunction is deprecated.
解决办法:在PHP8.2之后如果还想要使用动态属性,则需要在类定义前添加一个注解: #[\AllowDynamicProperties] .

    <?php
    class DefaultBehaviour { }
    
    #[\AllowDynamicProperties]
    class ClassAllowsDynamicProperties { }
    
    $o1 = new DefaultBehaviour();
    $o2 = new ClassAllowsDynamicProperties();
    
    $o1->nonExistingProp = true;
    $o2->nonExistingProp = true;
    p = true;
    ?>

结果:Deprecated: Creation of dynamic property DefaultBehaviour::$nonExistingProp is deprecated in file on line 10
没有注解的类不能添加动态属性!

----------
笔记7

**mail at mkharitonov dot net** 20-Feb-2014 11:20

Some comparisons of PHP and JavaScript closures.  
  PHP和JavaScript闭包的一些比较。(因为这个笔记是2014年的,所以有些内容已经过时,作为参考学习)
  
=== Example 1 (passing by value) ===  
PHP code:  
闭包内部作用域与外部隔绝

      <?php  
        $aaa = 111;  
        $func = function() use($aaa){ print $aaa; };  
        $aaa = 222;  
        $func(); // Outputs "111"  
        ?> 
//解释:通过use传递到匿名函数内部的变量,会变成静态成员,从此与外部的原始变量无关,是另外独立存在的,不会因为外面的aaa变化而变化.
    
Similar JavaScript code:  
JS想实现内部作用域隔绝的效果,需要使用双层函数的语法结构
  
    < script type="text/javascript">  
    var aaa = 111;  
    var func = (function(aaa){ return function(){ alert(aaa); } })(aaa);  
    aaa = 222;  
    func(); // Outputs "111"  
    < /script>

解释:JS中,()()双括号的语法结构,第一个括号括起来的会被认为是一个函数,第二个括号里面的会被认为是参数,然后立即调用前面括号里的函数;这就是JS 的IIFE(Immediately-invoked function expression):立即执行函数表达式.
括号内的是一个匿名函数,里面返回的还是匿名函数(所谓的双层函数),但在js中,变量是全局的,所以不需要像php那样使用use 来传递aaa进里面那个匿名函数.js可以直接使用闭包外面的变量.这种第二层的匿名函数,最后返回,里面也会存在一个aaa,而且这个aaa和外面的aaa脱离关系,独立存在.所以结果输出的还是之前的111 .
之所以出现这种写法是因为ES6之前的JS是没有块作用域的概念,全部变量都'全局'的.但这种双层函数却可以作用变量隔离的效果.这只能说是JS设计上一个奇怪的地方.
  
  
Be careful, following code is not similar to previous code:  

    < script type="text/javascript">  
    var aaa = 111;  
    var bbb = aaa;  
    var func = function(){ alert(bbb); };  
    aaa = 222;  
    func(); // Outputs "111", but only while "bbb" is not changed after function declaration  
     < /script> 

解释:这种直接赋值匿名函数给变量的(单层函数),跟上面的双层函数的结果不一样,这种匿名函数只有一层.运行时,里面的变量bbb是查找当前作用域找到的.aaa变了,但bbb的值是从aaa之前就复制过来了.跟aaa没有关系了.结果依然是111
  
// And this technique is not working in loops:  这种技巧在循环中无效

	< script type="text/javascript">  
    var functions = [];  
    for (var i = 0; i < 2; i++)  
    {  
	    var i2 = i;  
	    functions.push(function(){ alert(i2); });  
    }  
    functions[0](); // Outputs "1", wrong!  
    functions[1](); // Outputs "1", ok  
    < /script> 

 解释:这种也是一层函数,是找当前作用域下的那个变量,弹出i2,而i2循环结束时最后的值是1,而js(es6之前的)是没有局部作用域这个概念的,i2在循环的外面依然存在,并且可以访问.所以找到的值都是1 . 
  
=== Example 2 (passing by reference) ===  
PHP code:  

    <?php  
    $aaa = 111;  
    $func = function() use(&$aaa){ print $aaa; };  
    $aaa = 222;  
    $func(); // Outputs "222"  
    ?>  
解释:通过引用传递,那么指向的是同一个内存地址,只要这个内存地址里面的值改变了,无论在哪里获取的都是改变后的值.
打印这个闭包,可以看到aaa在静态数组里面,但也有一个&的引用符号

    ject(Closure)#2 (1) {
      ["static"]=>
      array(1) {
        ["aaa"]=>
        &int(222)
      }
    }
     
Similar JavaScript code:  

    < script type="text/javascript">  
    var aaa = 111;  
    var func = function(){ alert(aaa); };  
    aaa = 222; // Outputs "222"  
    func();  
    < /script>

解释:就是上面说过的,js的变量是全局的,js没有局部作用域概念,即使在匿名函数内部,也可以直接使用外部的变量.
这个例子有点太古老, 但也有一些价值.主要是以后遇到这样的写法的js需要注意.但是现在ES6的js已经增加了局部作用域和类的概念.所以这种双层函数的写法越来越少人用了.

---------
笔记8

**derkontrollfreak+9hy5l at gmail dot com** 09-Jan-2014 04:41
Beware that since PHP 5.4 registering a Closure as an object property that has been instantiated in the same object scope will create a circular reference which prevents immediate object destruction:

注意，从PHP 5.4起将Closure注册为在同一对象范围内实例化的对象属性，将创建一个循环引用，从而防止立即销毁对象:
这句话讲了半天不知道在说什么,我们直接举实例来说明区别:
例子:

    <?php
    
    class Test
    {
        private $closure;
    
        public function __construct()
        {
            $this->closure = function () {};
    	//var_dump($this->closure);
        }
    
        public function __destruct()
        {
            echo "destructed\n";
        }
    }
    
    new Test();
    echo "finished\n";
    /*
     * Result in PHP 5.3:
     * destructed
     * finished
     *
     * Result since PHP 5.4:
     * finished
     * destructed
     */

作者已经给出结果,可以看出在5.3中是先销毁类(触发析构方法__destruct),然后再echo,Php5.4就会先echo后面才销毁类.
我们首先要知道PHP销毁内容的时机:1程序结束时,会把全部变量销毁,变量所指向的对象也会被销毁. 2如果一个对象没有任何变量指向它,那它就无法访问,没有存在的意义了,此时php会立刻销毁这个对象.
之所以会有这样的区别的原因是:PHP5.4起,会自动在closure类里面生成一个this属性指向当前类,而PHP5.3则没有.
代码显示:
在__construct中执行: `var_dump($this->closure);`
PHP5.4的结果: 实际我用的是php8.2.6

    object(Closure)#2 (1) {
      ["this"]=>
      object(Test)#1 (1) {
        ["closure":"Test":private]=>
        *RECURSION*
      }
    }

PHP5.3.29的结果:专门去php官网下载了这个版的php

    object(Closure)#2 (0) {
    }

所以,PHP5.3在`new Test()`后,并没有任何指向对象的变量.所以对象被立即销毁.不用等到程序结束;而PHP5.4,则因为在闭包里面有个`$this`变量指向了对象.所以不会立即销毁,会在程序结束时才销毁.
我们现在知道了原因,那么要解决这个销毁时机不一样的问题,其实也很简单,
方法1:存在一个变量,指向对象.这样就不会立即销毁,而是在最后程序结束才销毁.
那么只需要 `$a = new Test();`
这样就符合有变量一直指向对象;
方法2:既然是因为有那个`$this`导致的不会立即销毁,那么我们就让这个`$this`不存在,
这样需要使用静态方法去生成匿名函数,应该在一个静态方法中生成匿名函数,因为在静态方法的作用域中是没有`$this`的(但很多时候,我们其实也需要在闭包里面使用到`$this`的,这样直接铲除`$this`,那么你想调用当前当前类其他属性和方法就不行了);
通过静态方法返回一个匿名函数:

    public function __construct()
    {
        $this->closure = self::createClosure();
    	var_dump($this->closure);
    }
    
    public static function createClosure()
    {
        return function () {
        };
    }
    
    此时的结果:
    object(Closure)#2 (0) {
    }
    destructed
    finished

可以看到:闭包里面没有this,现在是先销毁,后echo.
方法1:保持一直有变量指向对象,让对象在最后程序结束才销毁,方法2:确保没有变量指向对象,让对象立即销毁 .这两种效果在php5.3和5.4里面是保持一致的.
虽然说这个先销后销的时机问题,通常没人会去注意到,也不会实际影响程序执行.但是也加深了我们对于闭包和php销毁机制的理解.

----------
笔记9

**cHao** 13-Dec-2013 11:42
匿名函数

In case you were wondering (cause i was), anonymous functions can return references just like named functions can. Simply use the & the same way you would for a named function...right after the `function` keyword (and right before the nonexistent name).  
如果您想知道（因为我想知道），匿名函数可以像命名函数一样返回引用。只需使用与命名函数相同的方式即可。就是在“function”关键字之后（以及不存在的名称之前）。  

php的引用的写法,就是在变量或者函数、对象等前面加上&符号,在PHP中引用的意思是：不同的名字访问同一个变量内容,引用传递的是内存地址,不是把变量的值另外复制一份,并没有新占用一块内存;
函数使用引用的话,返回的结果直接是函数返回值的'引用'.如果`$a`是接收结果的变量,函数`return $b` ,效果相当于`$a = &$b` ;
例子:

    <?php  
    $value = 0;  
    $fn = function &() use (&$value) { return $value; }; //匿名函数因为没有名称,直接在参数括号前加&,而命名函数是在函数名前加&,这里use的变量也是引用传递,是借用了$vaule在这个作用域内都有效的性质,如果不用&$value,那么匿名函数内部的$value只是外部的$value的复制,之间就没有联系了.就不能通过++$x来影响到外部的$value了 
      
    $x =& $fn();  //这里的效果等同于$x = &$vaule
    var_dump($x, $value); // 'int(0)', 'int(0)'  
    ++$x;  //++$x,等同于++$vaule
    var_dump($x, $value); // 'int(1)', 'int(1)'

这里表明:`$x`是可以影响到`$value`的.因为`$x`引用了`$value`,它们指向的内存地址是同一个.这里例子的过程巧妙在:`use (&$value)`把`$value`的通过引用的方式传入匿名函数内部,然后匿名函数又通过函数返回引用的方式把`$value`又传出来了.整个过程都没有创建`$value`的值的副本.

上面这个例子可以使用静态变量的方式改写成:

    ?php
    $fn = function &() {
        static $value = 0;
        var_dump($value);
        return $value;
    };
    
    $x =&$fn();
    ++$x;
    $fn();
    ++$x;
    $fn();
    ++$x;
    $fn();
    结果:
    int(0)
    int(1)
    int(2)
    int(3)
    如果改成是: 
    $x =$fn();
    ++$x;
    $fn();
    ++$x;
    $fn();
    ++$x;
    $fn();
    var_dump($x);
    
    那么结果是:
    int(0)
    int(0)
    int(0)
    int(0)
    int(3)
    
    可以看到,如果是返回引用,那么$x就是静态变量$value,对于$



可以看到,如果是返回引用,那么`$x`就是静态变量`$value`,对于`$x`的改变,也会作用于`$value`.如果只是普通的传值,`$x`加了3次变成了3,`$value`也不会变化依然是0.
要使用函数返回引用,必须在声明函数时在函数名前面加&来表明函数可以返回引用.然后在调用时,接收结果的变量也必须在函数名前加&来表明要接收的是引用.
如果声明函数时不加&,而在接收时加&,会报错:PHP Notice:  Only variables should be assigned by reference (只有变量才能通过引用赋值)
如果声明函数时加&,而在接收时不加&,那么这次函数的返回值就是值传递,跟普通调用是一样的.


---------
笔记10

**mike at borft dot student dot utwente dot nl** 24-Jan-2012 02:46
Since it is possible to assign closures to class variables, it is a shame it is not possible to call them directly. ie. the following does not work:
由于可以为类变量分配闭包，所以很遗憾不能直接调用它们。即以下情况不起作用：


    <?php
    class foo {
    
      public test;
    
      public function __construct(){
        $this->test = function($a) {
          print "$a\n";
        };
      }
    }
    
    $f = new foo();
    
    $f->test();
    ?>
    
    However, it is possible using the magic __call function:
    <?php
    class foo {
    
      public test;
    
      public function __construct(){
        $this->test = function($a) {
          print "$a\n";
        };
      }
    
      public function __call($method, $args){
        if ( $this->{$method} instanceof Closure ) {
          return call_user_func_array($this->{$method},$args);
        } else {
          return parent::__call($method, $args);
        }
      }
    }
    $f = new foo();
    $f->test();
    ?>
    it

Hope it helps someone ;)



这里例子也是上面有说过,你不能直接像调用普通函数那种调用匿名函数,而是需要通过call_user_func_array变通地调用.
要么就是在外部用另外一个变量复制类里面的变量,然后再执行.
比如:


    $a = $f->test;
    $a();


----------

笔记11
这个例子是行不通的,会报错:call_user_func() expects parameter 1 to be a valid callback, function '{closure}' not found or invalid function name ,可能以前旧版本的php可行?还是说作者自己并没有运行过,因为他写也只是一个框架,不是实际代码.
在php5.3.29 和8.2.6都是一样的报错.

**gabriel dot totoliciu at ddsec dot net** 19-Jul-2010 10:56

If you want to make a recursive closure, you will need to write this:  
如果你想做一个递归闭包，你需要写:

    $some_var1="1";  
    $some_var2="2";  
      
    function($param1, $param2) use ($some_var1, $some_var2)  
    {  
      
    //some code here  
      
    call_user_func(__FUNCTION__, $other_param1, $other_param2);   //这里并不能运行,因为__FUNCTION__获取到的值是closure,而这个名字并不代表任何可调用的函数!会报错,
     //__FUNCTION__ : 魔术常量，获取当前的函数名
      
    //some code here  
      
    }  

  
If you need to pass values by reference you should check out  
如果需要通过引用传递值,你就需要看下面2个函数的说明文档,
  
[http://www.php.net/manual/en/function.call-user-func.php](http://www.php.net/manual/en/function.call-user-func.php)  
[http://www.php.net/manual/en/function.call-user-func-array.php](http://www.php.net/manual/en/function.call-user-func-array.php)  
  
If you're wondering if $some_var1 and $some_var2 are still visible by using the call_user_func, yes, they are available.
如果你想知道$some_var1 和 $some_var2 通过使用call_user_func后,是否依然是可见的,对!他们依然是可访问的,可获得的.
所以如果想递归调用匿名函数,还是选择用之前的方法: 使用use关键字并传递对函数本身的引用

-----------
笔记12
**kdelux at gmail dot com** 14-May-2010 08:55

Here is an example of one way to define, then use the variable ( $this ) in Closure functions.  The code below explores all uses, and shows restrictions.
下面是一个定义方法的示例，然后在闭包函数中使用变量（$this）。下面的代码探索了所有用途，并显示了限制。

The most useful tool in this snippet is the requesting_class() function that will tell you which class is responsible for executing the current Closure(). 
这个片段中最有用的工具是requesting_class（）函数，它将告诉您哪个类负责执行当前的Closure（）。

这个例子是十几年前写的,对于新版的php已经不适用,很多结果已经不同,报错的内容也不同.例子的作者说可以运行的,但在我运行的php5.3和8.2一些代码已经不能运行了.所以这个例子的作用不大,只有某些知识点值得参考了解,大部分已经过时了,对于新版php来说是错误的.
例子:


    <?php
     
        function requesting_class()
        { //debug_backtrace: 产生一条回溯跟踪,就是一个数组,里面是调用的代码,在哪个文件,哪一行,哪个对象,哪个函数,有什么参数等信息
            foreach(debug_backtrace(true) as $stack){
                if(isset($stack['object'])){
                    return $stack['object'];
                }
            }
            
        }
        
        
        class Person
        {
            public $name = '';
            public $head = true;
            public $feet = true;
            public $deflected = false;
            
     //当尝试以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用。
            function __invoke($p){ return $this->$p; }
     /*
        在直接输出对象引用的时候，就不会产生错误，而是自动调用了__tostring()方法,输出__tostring()方法中返回的字符串
        通俗来说就是 对象一般是使用print_r() 或 var_dump() 来打印访问
        但对于一般闲的人来说直接 使用 echo 输出对象时，必定会报错的，原因是对象无法使用echo的
        */
            function __toString(){ return 'this'; } // test for reference
            
            function __construct($name){ $this->name = $name; }
            function deflect(){ $this->deflected = true; }
            
            public function shoot()
            { // If customAttack is defined, use that as the shoot resut.  Otherwise shoot feet
                if(is_callable($this->customAttack)){
                    return call_user_func($this->customAttack);
                }
                
                $this->feet = false;
            }
        }
    
        $p = new Person('Bob');
    
        
        $p->customAttack = 
                    function(){
                        
                        //测试1
        //echo $this; // Notice: Undefined variable: this ,Using $this when not in object context,这里php认为当前环境(上下文)并不是在类中,不能使用$this,或者也可以认为不存在$this
        //测试2
        //$this = new class(); // FATAL ERROR ,syntax error ,没有这个写法的,创建匿名类是下面的写法
        //$this = new class  { }; //Fatal error:  Cannot re-assign $this ,不能重新分配$this
     
        //测试3
        // Trick to assign the variable '$this' ,对分配变量$this的诡计,但这个诡计会失败
        //extract:从数组中将变量导入到当前的符号表,把一个关联数组。此函数会将键名当作变量名，值作为变量的值。快捷自动创建一堆变量.但对于一些不可信的内容(用户输入的),千万要小心!
        //这里会报:Cannot re-assign $this ,不能重新分配$this的错误
        //extract(array('this' => requesting_class())); // Determine what class is responsible for making the call to Closure 确定哪个类负责调用闭包
        
        //测试4
        //var_dump($this);  // Passive reference works ,报错:Using $this when not in object context
        
        //测试5 报错:Using $this when not in object context
        //var_dump($$this); // Added to class:  function __toString(){ return 'this'; } 这里直接是无法运行的.但是如果在类中,就可以.执行流程:$后面跟着的是字符串,$$this会造成Php认为你在把对象转为字符串,然后触发调用__toString方法,然后返回字符串this,然后和前面的$巧妙地组合成$this,最终打印出这个实例.这个曲折的写法有点搞笑.
        
        //测试6 php8.2.6报错:Using $this when not in object context ,php5.3.29报错: Function name must be a string ,因为在5.3版本$this 认为是 null,使用$this并不会报错!
        /*     $name = $this('name'); // Success ,这个人说成功,应该是他用的php版本旧?毕竟这个笔记是13年前写的了
        echo $name;            // Outputs: Bob ,上面就报错了,所以这里的结果已经不成立了
        echo '<br />'; */
     
        //测试7 报错:Using $this when not in object context
        //echo $$this->name;
        //var_dump($$this->name) ;
    
        //测试8 报错:Using $this when not in object context.  php5.3报错: call_user_func_array() expects parameter 1 to be a valid callback, first array member is not a valid class name or object ,因为这里$this是null
        //call_user_func_array(array($this, 'deflect'), array()); // SUCCESSFULLY CALLED
    
        //测试9 报错:Using $this when not in object context.
        //$this->head = 0; //** FATAL ERROR: Using $this when not in object context
        //$$this->head = 0;  // Successfully sets value 
                        
                    };
     
        print_r($p);
        
        $p->shoot();
        
        print_r($p);
    
        
        die();
    
    ?>

debug_backtrace得到的数组内容是这样的:


    array(7) {
        ["file"]=>
        string(81) "C:\......\25.php"
        ["line"]=>
        int(133)
        ["function"]=>
        string(9) "{closure}"
        ["class"]=>
        string(6) "Person"
        ["object"]=>
        object(Person)#1 (5) {
          ["name"]=>
          string(3) "Bob"
          ["head"]=>
          bool(true)
          ["feet"]=>
          bool(true)
          ["deflected"]=>
          bool(false)
          ["customAttack"]=>
          object(Closure)#2 (0) {
          }
        }
        ["type"]=>
        string(2) "->"
        ["args"]=>
        array(0) {
        }

参数可以有:


    debug_backtrace()/debug_backtrace(1)/debug_backtrace(true) - show all options
    debug_backtrace(0)/debug_backtrace(false) - exlude ["object"] 排除对象
    debug_backtrace(2) - exlude ["object"] AND ["args"]  排除对象和参数

排除掉一些内容后,返回的数据就没那么大,节省内存,有时候层次多的话,返回的内容非常多,数组非常巨大.而且有时候我们想要的并不是对象或参数的数据.


----------
笔记13

**rob at ubrio dot us** 25-Nov-2009 10:20

You can always call protected members using the __call() method - similar to how you hack around this in Ruby using send.
您可以始终使用__call（）方法调用受保护的成员，类似于您在Ruby中使用send破解此问题的方法。(我也不懂Ruby语言)
但这个例子跟匿名函数并没有什么功能上的关系,这个例子主要是做了一个魔术方法__call的实现,然后使实例可以在外部直接调用protected的方法.

    <?php
    
    class Fun
    {
     protected function debug($message)
     {
       echo "DEBUG: $message\n";
     }
    
     public function yield_something($callback)
     {
       return $callback("Soemthing!!");
     }
    
     public function having_fun()
     {
       $self =& $this;
       return $this->yield_something(function($data) use (&$self)
       {
         $self->debug("Doing stuff to the data");
         // do something with $data
         $self->debug("Finished doing stuff with the data.");
       });
     }
     // Ah-Ha!
    
     public function __call($method, $args = array())
     {
       if(is_callable(array($this, $method)))
         return call_user_func_array(array($this, $method), $args);
     }
    
    }
    
    $fun = new Fun();
    echo $fun->having_fun();
    ?>

结果:
DEBUG: Doing stuff to the data
DEBUG: Finished doing stuff with the data.

这个例子里面通过匿名函数曲折地调用了protected方法debug().暂时不理解为什么要这么曲折地去调用,明明可以在having_fun中直接调用(`$this->debug('having_fun');`)的.却偏偏通过`$self` 引用`$this`,然后再通过调用另外一个方法来调用匿名函数,还使用use传`$this`进匿名函数,最终调用了debug().非常复杂.不清楚为什么要这样写,因为不这样写就跟匿名函数无关了啊?
这个例子最重要的是那个__call方法(魔术方法的其中一个)的实现,通过这样的实现,本来不能直接调用的debug(),变成了可以直接调用;
如果没有__call方法,直接调用`$fun->debug('123');`,会报错:Call to protected method Fun::debug() from global scope :直接在全局作用域调用受保护的方法,这是不允许的,因为受保护的属性/方法,规定只能在类的内部调用.
但是加上了这个__call方法后,php在找不到方法的时候,最后就会尝试调用这个__call方法,然后此时就是在内部了,可以调用受保护的方法了.检查之后发现debug()是callable的,然后进行了调用.

我们添加 2个方法:

    public function selfDebug($message)
        {
    
            self::debug($message);
    
        }
    
    public function thisDebug($message)
       {
    
           $this->debug($message);
    
       }
    
    
    $fun->debug('123');
    $fun->selfDebug('slef');
    $fun->thisDebug('$this');

结果:
DEBUG: 123
DEBUG: slef
DEBUG: $this

我们发现新加的两个方法都是可以调用 protected method Fun::debug() 的;也就是说外部要使用protected的方法,需要套一层壳!
这种壳是专用的,一个protected方法就要对应套一个壳方法.但是按照例子中的__call()方法的实现,就可以实现不用写壳,而直接自动调用protected的方法.
顺便说一下:__call是用于调用不可访问的方法,要调用不可访问的属性,使用__get() ;
一个简单的__get实现:

    public function __get($name){
            if(isset($this->$name)){
                return $this->$name;
            }else{
                return new Exception('没有定义的属性'.$name);
            }
        }


-----------

笔记14

**Anonymous** 03-Aug-2009 02:50

If you want to check whether you're dealing with a closure specifically and not a string or array callback you can do this:
如果你想明确地想知道你正常处理的是一个闭包,而不是一个字符串或者数组,你可以这样检测:

    <?php
	    $isAClosure = is_callable($thing) && is_object($thing);
    ?>

解释:闭包本身是一个特殊的存在,php使用closure类来实现闭包,所以如果你创建了一个匿名函数,那么它既是一个对象实例,也是可调用的.闭包具有这种双重性!这个非常特别!所以可以拿来做判断一个变量是否是一个闭包.

---------
笔记15

**a dot schaffhirt at sedna-soft dot de** 19-Jun-2009 02:55
When using anonymous functions as properties in Classes, note that there are three name scopes: one for constants, one for properties and one for methods. That means, you can use the same name for a constant, for a property and for a method at a time.
当在类中使用匿名函数作为属性时，请注意有三个名称作用域：一个用于常量，一个用于属性，另一个用于方法。这意味着，一次可以对一个常量、一个属性和一个方法使用相同的名称。

Since a property can be also an anonymous function as of PHP 5.3.0, an oddity arises when they share the same name, not meaning that there would be any conflict.
由于属性也可以是PHP 5.3.0中的匿名函数(在php8.2.5中这个例子依然相同结果)，因此当它们共享相同的名称时会出现有点奇特的情况，这并不意味着会有任何冲突。

    <?php
        class MyClass {
            const member = 1;
            public $member = 2;
            
            public function member () {
                return "method 'member'";
            }
            
            public function __construct () {
                $this->member = function () {
                    return "anonymous function 'member'";
                };
            }
        }
    
        $myObj = new MyClass();
    
        var_dump(MyClass::member);  // int(1)
    
        var_dump($myObj::member);  // int(1)
        var_dump($myObj->member);   // object(Closure)#2 (0) {}
        var_dump($myObj->member()); // string(15) "method 'member'"
       
        $myMember = $myObj->member;
        var_dump($myMember());      // string(27) "anonymous function 'member'"
    ?>

结果:

    int(1)
    int(1)
    object(Closure)#2 (1) {
      ["this"]=>
      object(MyClass)#1 (1) {
        ["member"]=>
        *RECURSION* //递归的.也就无限套娃,循环引用
      }
    }
    string(15) "method 'member'"
    string(27) "anonymous function 'member'"

解释:常量,属性,方法 这3个东西可以使用相同的名字.那么使用的时候如何区分呢.简单来说,常量所以可以直接使用双冒号::(双冒号操作符：即作用域限定操作符Scope Resolution Operator可以访问静态、const和类中重写的属性与方法),无论是类名直接::常量,还是实例::常量, 指向的都是同一个内存地址.可以看做是静态的.
用箭头运算符->调用时,左侧必须是实例,右侧名字后面如果有括号(括号表示要进行调用),表示这个名字代表一个方法,没有则代表是一个属性.
MyClass::member,符合要求的只有是const member ;
$myObj::member ,和上面指向的是同一个东西;
$myObj->member,符合要求的是一个属性,public $member,而这个属性的值,已经在构造函数中变成了一个匿名函数;
$myObj->member(),要调用一个方法,也就是public function member;
$myMember(),myMember复制于属性member,属性member是一个匿名函数,也就是调用这个匿名函数,自然得到的是匿名函数的返回值:anonymous function 'member'


That means, regular method invocations work like expected and like before. The anonymous function instead, must be retrieved into a variable first (just like a property) and can only then be invoked.
这意味着，常规方法调用的工作方式与预期和以前一样。然而匿名函数必须首先赋值到一个变量中（就像属性一样），然后才能调用。意思就是说如果那个属性的值是匿名函数,但你还想调用它,那么你只能是用另外一个变量复制(直接复制)它的值,然后再用那个变量进行调用.如果你直接调用(写法是$myObj->member()),实际调用的是同名方法,不是匿名函数.即使你不存在同名方法,也不能这样调用,会报错:Call to undefined method MyClass::member() ,php不会去判断是否存在一个叫member的属性而且还是callable的.

这里例子主要是要记住一个类里面可以有3个名字相同的,但名称作用域不同的'数据' ;还有就是要记住如何区分要使用的是哪一个数据.通过箭头符和后面是否带有括号来区分.最后是怎样调用保存在属性中的匿名函数.

----------

**好了这里就是Php官方手册关于匿名函数的说明的页面的,全部内容的解释.希望可以帮到你了解Php的匿名函数.**
