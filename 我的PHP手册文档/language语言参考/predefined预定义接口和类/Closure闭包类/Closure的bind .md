## Closure::bind

(PHP 5 >= 5.4.0, PHP 7, PHP 8)

Closure::bind — 用特定的绑定对象和类作用域复制闭包。(这个翻译完全无法理解说的是什么,说人话就是:参数传入对象和类,然后复制原本的闭包,返回一个组合后的新的闭包,具体需要看下面的例子才能理解)

### 说明

    public static Closure::bind(Closure $closure, ?object $newThis, object|string|null $newScope = "static"): ?Closure

这个方法是 `Closure::bindTo()` 的静态版本。查看它的文档获取更多信息。

### 参数

`closure`
需要绑定的匿名函数。必须的.要求类型:闭包类.

`newThis`
需要绑定到闭包的对象(绑定为闭包内部的`$this`)，或者 **`null`** 创建未绑定的闭包。可选的:但没有的话,不能为空,因为至少需要2个参数(Closure::bind() expects at least 2 arguments),需要写个 null 占位.要求类型:对象实例

`newScope`
想要绑定给闭包的类作用域，或者 'static' 表示不改变。如果传入一个对象，则使用这个对象的类型名。 类作用域用来决定在闭包中 $this 对象的 私有、保护方法 的可见性。 不允许内置类（的对象）作为参数传递。
可选的,默认值是字符串"static",你也可以传:类的实例/类的名字/null . 

### 返回值

返回一个新的 Closure 对象(也就是返回一个新的经过加工后的闭包类)，失败时返回 **`null`**。 

### 示例

**Example #1 **Closure::bind()** 示例**

    <?php  
	    class A {  
		    private static $sfoo = 1;  
		    private $ifoo = 2;  
	    }  
	    $cl1 = static function() {  
		    return A::$sfoo;  
	    };  
	    $cl2 = function() {  
		    return $this->ifoo;  
	    };  
	     //bind完之后,返回一个新的匿名函数,并没有改变/销毁原匿名函数
	    $bcl1 = Closure::bind($cl1, null, 'A');  //把A类的作用域和cl1绑定,这样cl1就能访问A的私有/受保护的资源了
	    $bcl2 = Closure::bind($cl2, new A(), 'A');  //把A的实例和A的作用域与cl2绑定,这样cl2内部的$this就是A的实例,同时也能访问A的私有/受保护的资源.这就好像匿名函数是写在A的内部一样!
	    echo $bcl1(), "\n";  
	    echo $bcl2(), "\n";  
    ?>

以上示例的输出：

1
2

----------

官方的这个例子1没有详细讲清楚各种区别,我们以这个例子为基础,改编后,来说明各种情况;

    <?php
    class A {
        private static $sfoo = 1;
        protected $ifoo = 2;
        public static $kfoo = 3;
        public  $mfoo = 4;
    }
    $cl1 = static function() {
        echo A::$sfoo, "\n";
        echo $this->ifoo, "\n";
        echo A::$kfoo, "\n";
        echo $this->mfoo, "\n";
    };

//第1种参数组合

    $bcl1 = Closure::bind($cl1); 
    //Uncaught ArgumentCountError:Closure::bind() expects at least 2 arguments

最少需要2个参数

//第2种参数组合
	
    $bcl1 = Closure::bind($cl1, null); //也就是$bcl1 = Closure::bind($cl1, null,'static');
	var_dump($bcl1); //打印: object(Closure)#4 (0) {} 返回值也是一个闭包
	echo A::$sfoo, "\n"; // 报错:Uncaught Error: Cannot access private property A::$sfoo,没有传入参数3,不能访问A的私有/受保护的资源
    echo $this->$ifoo, "\n"; //报错:Using $this when not in object context ,原因:bind的参数2没有传一个实例过去.闭包内部不存在$this.并且$cl1是静态匿名函数,内部本来就是不能有$this的.
    echo A::$kfoo, "\n"; // 输出:3,$kfoo是A类的公开的静态资源,本身就可以在任何地方直接访问,不管有没有参数3.
    echo $this->$mfoo, "\n"; // //报错:Using $this when not in object context.
 

//第3种参数组合

    $bcl1 = Closure::bind($cl1, null, 'A'); //把A的作用域绑定到cl1中,让cl1可以访问A的全部资源(包括私有/受保护的资源);
    
    $bcl1(); // 结果如下:
    echo A::$sfoo, "\n"; // 输出:1,参数3有传A进来,那么在闭包内部就可以使用A作用域里面的全部资源,这就是与参数组合1的区别
    echo $this->$ifoo, "\n"; //报错:Using $this when not in object context
    echo A::$kfoo, "\n"; // 输出:3
    echo $this->$mfoo, "\n"; // //报错:Using $this when not in object context


//第4种参数组合

    $bcl1 = Closure::bind($cl1, new A(), 'A'); //把A的实例绑到了闭包里面,并且作为闭包里面的$this,同时也把类型:A 绑了进去.
    //这里在绑定的过程中就直接报错了:Cannot bind an instance to a static closure,不能把一个实例绑定到一个静态闭包!
    //原因:一个静态的匿名函数里面是没有(不能有)$this的!!!也就是如果你的闭包本身是静态的,那么bind的第二个参数必须为null,
   

//第5种参数组合

    //需要创建一个非静态的匿名函数
    $cl2 = function() {
        echo A::$sfoo, "\n";
        echo $this->ifoo, "\n";
        echo A::$kfoo, "\n";
        echo $this->mfoo, "\n";
    };
    
    $bcl2 = Closure::bind($cl2, null, 'A'); 
    
    var_dump($bcl2); //打印: object(Closure)#4 (0) {}
    $bcl2(); //结果如下:
    
    echo A::$sfoo, "\n"; //输出:1
    echo $this->ifoo, "\n"; //报错:Using $this when not in object context,因为没有传参数2
    echo A::$kfoo, "\n"; //输出:3
    echo $this->mfoo, "\n"; //报错:Using $this when not in object context

//第6种组合

    $bcl2 = Closure::bind($cl2, new A(), 'A'); 
    var_dump($bcl2); //打印:
    /*
    object(Closure)#4 (1) {
	  ["this"]=>
	  object(A)#3 (2) {
	    ["ifoo":protected]=>
	    int(2)
	    ["mfoo"]=>
	    int(4)
	  }
	}
    */
此时我们可以看到这个闭包里面增加了this变量,指向的就是A的实例,此时我们就可以在闭包里面使用$this了;

    echo A::$sfoo, "\n"; //输出1,正常
    echo $this->ifoo, "\n"; //输出2,正常
    echo A::$kfoo, "\n"; //输出3,正常
    echo $this->mfoo, "\n";  //输出4,正常

全部正常! 

//第7种组合

    $bcl2 = Closure::bind($cl2, new A(), null); 
    var_dump($bcl2); //打印:
    /* 跟组合6是一样
      object(Closure)#4 (1) {
	  ["this"]=>
	  object(A)#3 (2) {
	    ["ifoo":protected]=>
	    int(2)
	    ["mfoo"]=>
	    int(4)
	  }
	}
    */
    echo A::$sfoo, "\n"; //报错:Cannot access private property A::$sfoo,原因:bind的第三个参数没有传A进来,不能访问A的私有/受保护的资源
    echo $this->ifoo, "\n"; //报错: Uncaught Error: Cannot access protected property A::$ifoo 官方说:类作用域(参数3,$newScope)用来决定在闭包中 $this 对象的私有、保护方法的可见性. 同时传入A的实例和A类(像组合6),才可以使用$this访问A的私有和受保护属性和方法;
    
    echo A::$kfoo, "\n"; //输出3,正常
    echo $this->mfoo, "\n";  //输出4,正常

//第8种组合

     $bcl1 = Closure::bind($cl1, null,null);
     //这种写法,参数3是null,这会导致$cl1如果本身已经有关联作用域的话,会被清除,它就失去了关联作用域.
     //有兴趣的可以用下面的笔记1的例子来做实验,进行双重bind,使用getClosureScopeClass(),就可以得知bind后的闭包的关联作用域的那个类.

总结: 你想在闭包中访问某个类的全部资源(私有/受保护的),就需要使用参数3传递这个类的名字或者实例.
你想在闭包中使用`$this`,那么参数1的原始匿名函数不能是静态的,然后在参数2传递实例.
如果你想在闭包中直接使用`$this->ifoo` ,`$ifoo`是私有/受保护的资源,那么3个参数你都需要传递.并且参数2和参数3都是指同一个类.

----------

### 参见

-   匿名函数
-   Closure::bindTo() - 用特定的绑定对象和类作用域复制闭包。

----------

### User Contributed Notes 用户笔记

笔记1

**potherca at hotmail dot com** 03-Dec-2014 08:22

If you need to validate whether or not a closure can be bound to a PHP object, you will have to resort to using reflection.  
  如果你需要验证一个闭包是否是可以被绑定到一个PHP对象,你需要借助于反射类.(反射类是一种可以获得类/函数各种信息数据的类,可以理解为:用于描述类的类)

    <?php  
      
    /**  
    * @param \Closure $callable  
    *  
    * @return bool  
    */  
    function isBindable(\Closure $callable)  
    {  
	    $bindable = false;  
	    $reflectionFunction = new \ReflectionFunction($callable);  //ReflectionFunction 内置类用于展示指定函数的各种信息 
	    //getClosureScopeClass: Returns the scope associated to the closure 返回与闭包关联的作用域
	    //getClosureThis: Returns this pointer bound to closure 这句话有3种翻译:1返回绑定到闭包的指针 /2返回本身的匿名函数/3返回 $this 指向，产生错误返回 null ,哪一种是正确的理解呢?
	    echo  "reflectionFunction", "\n";
	    var_dump($reflectionFunction);
	   
	    echo  "getClosureScopeClass", "\n";
	    var_dump($reflectionFunction->getClosureScopeClass());
	   
	    echo  "getClosureThis", "\n";
	    var_dump($reflectionFunction->getClosureThis());

	    if (   $reflectionFunction->getClosureScopeClass() === null   || $reflectionFunction->getClosureThis() !== null   ) {  
		    $bindable = true;  
	    }  
      
	    return $bindable;  
    }  
    ?>

写一个例子展示一下都获取到了什么东西:
		
		//拿来被绑定的类
    	class  A {
		    public  $mfoo = 1;
		   }
		//用来绑定的匿名函数
		$a = function(){var_dump('a');};
		
		isBindable($a);
		得到结果:
		reflectionFunction
		object(ReflectionFunction)#2 (1) {
		  ["name"]=>
		  string(9) "{closure}"
		}
		getClosureScopeClass
		NULL
		getClosureThis
		NULL
		bool(true)
		总结:一个普通的匿名函数,得到它的关联作用域(ClosureScopeClass)是没有的,它的$this指向也是没有的.但如果匿名函数是在类中,那么就会有$this;

	    如果我们进行bind过之后
	    $binded = Closure::bind($a, new  A(), 'A');
		isBindable($binded);
		得到结果:
		reflectionFunction
		object(ReflectionFunction)#4 (1) {
		  ["name"]=>
		  string(9) "{closure}"
		}
		getClosureScopeClass
		object(ReflectionClass)#5 (1) {
		  ["name"]=>
		  string(1) "A"
		}
		getClosureThis
		object(A)#2 (1) {
		  ["mfoo"]=>
		  int(1)
		}
		bool(true)
									
		总结:如果一个匿名函数进行完整的bind()后,再通过这个匿名函数的反射类,那么就可以通过getClosureScopeClass得到它的关联作用域的类的反射类,和用getClosureThis得到它的$this指向的那个类.

但这个例子说是判断是否可以进行绑定,但实际上,PHP并没有限制你可以bind多少次, 你可以对一个闭包1进行bind后,得到一个新的闭包2,然后闭包2又进行bind,得到闭包3,如此一直延续下去都是可以的.
这个用户的说法应该是指:如果你知道一个闭包它没有关联作用域,你就可以bind. 或者它有`$this` ,你也可以bind.
我们对一个闭包进行bind的操作,只不过是切换它的关联作用域和$this指向而已.这个切换我们可以进行无数次.

      


----------
笔记2

**Vincius Krolow** 14-Dec-2012 12:08

With this class and method, it's possible to do nice things, like add methods on the fly to an object.  
  通过这个类和方法, 可以做一些好玩的事，比如向对象动态添加方法。
MetaTrait.php  

    <?php  
    trait MetaTrait  
    {   
	    private $methods = array();  
      
	    public function addMethod($methodName, $methodCallable)  
	    {  
		    if (!is_callable($methodCallable)) {  
			    throw new InvalidArgumentException('Second param must be callable');  //如果是不可调用的就报错
		    }  
		    $this->methods[$methodName] = Closure::bind($methodCallable, $this, get_class());  
		    //把当前的$this绑为methodCallable内部的$this,get_class()返回对象的类名,让methodCallable得到当前类的作用域,这里参数3必须传入类名,否则匿名函数无法调用私有变量
	    }  
	     //如果调用了一个类不存在的方法,最后会调用这个__call魔术函数
	    public function __call($methodName, array $args)  
	    {  
		    if (isset($this->methods[$methodName])) {  
		    //去methods数组里面查找对应名称的元素,也就是方法名.因为之前放入数组前已经确保是callable,所以这里不需要判断,直接调用即可
			    return call_user_func_array($this->methods[$methodName], $args);  
		    }  
      
		    throw RunTimeException('There is no method with the given name to call');  
		}  
    }  
    ?>  
    /////////////////////
      
    test.php  
    <?php  
    require 'MetaTrait.php';  //引用这个文件
      
    class HackThursday {  
	    use MetaTrait;  //使用这个代码片段,等于是在这里写了MetaTrait.php里面的代码
      
	    private $dayOfWeek = 'Thursday';  
      
	}  
      
    $test = new HackThursday();  
    $test->addMethod('when', function () {  
	    //这里是类的外部定义的匿名函数,首先没有$this,其次作用域也没有关联 HackThursday 类的作用域
	    return $this->dayOfWeek;  
    });  
      
    echo $test->when();  
      结果: Thursday
    ?>

