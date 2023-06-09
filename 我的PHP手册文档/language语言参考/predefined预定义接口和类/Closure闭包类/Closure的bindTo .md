## Closure::bindTo 

(PHP 5 >= 5.4.0, PHP 7, PHP 8)

Closure::bindTo — Duplicates the closure with a new bound object and class scope .用特定的绑定对象和类作用域复制闭包。 (这句翻译依然令人摸不着头脑.可以翻译成:原闭包携带着新的绑定对象和类作用域去复制成新闭包)

----------

### 说明 

    public Closure::bindTo(?object $newThis, object|string|null $newScope = "static"): ?Closure

Create and return a new [anonymous function](https://www.php.net/manual/en/functions.anonymous.php) with the same body and bound variables as this one, but possibly with a different bound object and a new class scope.
创建并返回一个 匿名函数，它与当前对象的函数体相同、绑定了同样变量(这样理解:匿名函数内部的代码是相同的,其实就是同一份代码)，但可以绑定不同的对象(匿名函数内部的`$this`指向的对象实例可以改(绑定)成其他实例)，也可以绑定新的类作用域(很可能两个`$this`是不同类的实例,此时也就需要同步修改类作用域,除非`$this`不需要访问私有/受保护的成员)。 

The “bound object” determines the value `$this` will have in the function body and the “class scope” represents a class which determines which private and protected members the anonymous function will be able to access. Namely, the members that will be visible are the same as if the anonymous function were a method of the class given as value of the `newScope` parameter.
"绑定的对象"(参数1)决定了函数体中的 $this 的取值(指向)，"类作用域"(参数2)代表一个类型(类的名字或类的实例)、决定了在这个匿名函数中是否能够调用newScope 类的 private 和 protected 的成员。也就是说，此时 $this 可以访问的成员(方法/属性等)，与 newScope 类的成员函数是相同的(等同于这个匿名函数创建在类的内部,也就是写在类的内部)。 

Static closures cannot have any bound object (the value of the parameter `newThis` should be **`null`**), but this function can nevertheless be used to change their class scope.
静态闭包不能有绑定的对象(静态函数是不能有$this的)（ newThis 参数的值应该设为 null,必须要有至少一个参数,不能直接不写,没有的话就写null）不过仍然可以用 bindTo 方法来改变它们的关联类作用域。 

This function will ensure that for a non-static closure, having a bound instance will imply being scoped and vice-versa. To this end, non-static closures that are given a scope but a **`null`** instance are made static and non-static non-scoped closures that are given a non-null instance are scoped to an unspecified class.
此函数确保对于非静态闭包，拥有绑定实例也意味着被限定作用域，反之亦然。为此，非静态闭包给定一个 null 实例的作用域可以使其变为静态，非静态无作用域的闭包给定一个非 null 的实例作用在一个非指定类。 
百度翻译:
此函数将确保对于非静态闭包，具有绑定实例将意味着具有作用域，反之亦然。为此，被赋予作用域但具有“null”实例的非静态闭包被设为静态的，而被赋予非null实例的非静止非作用域闭包的作用域被设为未指定的类。
这段话的什么意思还不明白,等后面弄明白了再回来翻译!!!!!!!!!!!

----------
### 提醒
If you only want to duplicate the anonymous functions, you can use [cloning](https://www.php.net/manual/en/language.oop5.cloning.php) instead.
如果你只是想要复制一个匿名函数，可以用 cloning  代替。 而不是用bind来再次生成一个匿名函数。

----------

### 参数 

newThis
必需,绑定给匿名函数的一个对象，或者 null 来取消绑定。 

newScope
可选,关联到匿名函数的类作用域，或者 'static' 保持当前状态。如果是一个对象，则使用这个对象的类型为新的类作用域。这会决定绑定的对象的 保护、私有成员 方法的可见性。不允许内部类（的对象）作为参数传递。 如果使用null,则会取消绑定


### 返回值 

返回新创建的 Closure 对象，或者失败时返回 null 。 

----------
### 示例 


Example #1 Closure::bindTo() 实例

    <?php
    
    class A {
        private $val;
        function __construct($val) {
            $this->val = $val;
        }
        function getClosure() {
            // 返回绑定到此对象和作用域的闭包
            return function() { return $this->val; };
        }
    }
    
    $ob1 = new A(1);
    $ob2 = new A(2);
    
    $cl = $ob1->getClosure();
    echo $cl(), "\n";
    $cl = $cl->bindTo($ob2);
    echo $cl(), "\n";
    ?>  


以上示例的输出类似于：


1
2

解释:参数1把`$ob2`绑给闭包,那么闭包里面的`$this`就变成了`$ob2`,那么输出的val是ob2的.

----------


### 参见 
•匿名函数
•Closure::bind() - 用特定的绑定对象和类作用域复制闭包。

----------

### User Contributed Notes 用户笔记

 笔记1
Olexandr Kalaidzhy 12-Feb-2022 11:56 

 Get all object vars without using Reflection:不使用反射类来获取对象的全部属性

    <?php
    declare(strict_types=1); //开启严格语法模式
    class A
    {
        private $foo = 'foo';
        protected $bar = 'bar';
        public $buz = 'buz';
    }
    
    function get_object_vars_all($object): array
    {
        if (!\is_object($object)) {
            throw new \InvalidArgumentException(sprintf('The argument should be an object, "%s" given.', get_debug_type($object)));
        }
   
        $closure = function () {
            return get_object_vars($this);
        };
    
        return $closure->bindTo($object, $object)();//这里参数2把object也传了过去,导致$this具有了类A的作用域,可以访问A的全部属性,包括私有和受保护的属性.如果不传参数2,那么两个输出的结果是一样的.
    
    }
    
    $a = new A();
    
    var_dump(get_object_vars($a));
    
    var_dump(get_object_vars_all($a));
   
    ?>


 The output: 

 

    array(1) {
       ["buz"]=>
       string(3) "buz"
     }
     array(3) {
       ["foo"]=>
       string(3) "foo"
       ["bar"]=>
       string(3) "bar"
       ["buz"]=>
       string(3) "buz"
     } 

 ----------
 笔记2
Anonymous 19-Apr-2018 03:31 


 If you want to unbind completely the closure and the scope you need to set both to null:
 如果您想完全解除对闭包和作用域的绑定，则需要将两者都设置为null

    <?php
    class MyClass
    {
        public $foo = 'a';
        protected $bar = 'b';
        private $baz = 'c';
       
        /**
    
         * @return array
    
         */
    
        public function toArray()
        {
            // Only public variables
            return (function ($obj) { return get_object_vars($obj);})
		            ->bindTo(null, null)($this);
        }
    }
    ?>

 In this example, only the public variables of the class are exported (foo).
在本例中，仅导出类的公开变量（foo）。
 If you use the default scope (->bindTo(null)) also protected and private variables are exported (foo, bar and baz).
如果你使用默认类作用域(也就是binTo只是参数1写null,参数2缺失),这样会依然导出私有和受保护的变量(foo, bar and baz)。
 It was hard to figure it out because there is nowhere mentioned in the documentation that you can use null as a scope. 
这很难弄清楚，因为文档中没有提到可以使用null作为参数2的值(作用域)的地方。(这点我已经在参数2的说明中给官方文档补上了)
 
 ----------
 笔记3
luc at s dot illi dot be 31-May-2016 04:44 


 Access private members of parent classes; playing with the scopes:
 通过使用类作用域来访问父类的私有成员:

    <?PHP
    #[AllowDynamicProperties] //允许动态添加属性
    //爷类
    class Grandparents{ private $__status1 = 'married'; //已婚}
    //父类
    class Parents extends Grandparents{ private $__status2 = 'divorced'; //离婚}
    //子类(我类)
    class Me extends Parents{ private $__status3 = 'single'; //单身狗 }
    
    $status1_3 = function()
    {
	    //无论哪种状态都是快乐的
        $this->__status1 = 'happy';
        $this->__status2 = 'happy';
        $this->__status3 = 'happy';
    
    };
    
    $status1_2 = function()
    {
        //单身狗没有体验过快乐
        $this->__status1 = 'happy';
        $this->__status2 = 'happy';
    };
    
    // test 1:
    $c = $status1_3->bindTo($R = new Me, Parents::class);            
    
    #$c();    // Fatal: Cannot access private property Me::$__status3
    //原因:虽然$this指向Me的实例,但关联类作用域却是父类的作用域,此时$this的也只能访问父类的私有成员.而无权访问Me的私有成员,此时对于$this来说__status3存在,但不能访问.
    
    // test 2:
    $d = $status1_2->bindTo($R = new Me, Parents::class);
    $d();
    var_dump($R);
    /*
    
    object(Me)#5 (4) {
    
      ["__status3":"Me":private]=>
    
      string(6) "single"
    
      ["__status2":"Parents":private]=>
    
      string(5) "happy"  //看这里,__status2值被改了.因为$this找到了__status2,然后还发现有权进行操作,因为当前作用域指向了Parents,可以对Parents的私有变量进行操作.
    
      ["__status1":"Grandparents":private]=>
    
      string(7) "married"
    
      ["__status1"]=>
    
      string(5) "happy" //由于Parents类作用域内并不存在的__status1,所以动态新增了这个属性.Grandparents类中也有__status1,但Parents类并不能访问到,上级类(父类)的同名属性会被下级类(子类)重写(替换)掉(子类有A属性就优先获取子类的A,没有的话,就尝试去访问父类的A,类似冒泡,一层层往上找)
    
    }
    
    */
    
    
    
    // test 3:
    
    $e = $status1_3->bindTo($R = new Me, Grandparents::class);    
    
    #$e(); // Fatal: Cannot access private property Me::$__status3
    //原因跟test 1 是一样的
    
    
    
    // test 4:
    
    $f = $status1_2->bindTo($R = new Me, Grandparents::class);    
    
    $f();
    
    var_dump($R);
    
    /*
    
    object(Me)#9 (4) {
    
      ["__status3":"Me":private]=>
    
      string(6) "single"
    
      ["__status2":"Parents":private]=>
    
      string(8) "divorced"
    
      ["__status1":"Grandparents":private]=>
    
      string(5) "happy" //此时我们可以看到关联类作用域是Grandparents,所以找到了Grandparents的__status1,进行了改写
    
      ["__status2"]=>
    
      string(5) "happy" //而在Grandparents的作用域中,并不存在__status2属性,所以这里就新增了__status2
    
    }
    
    */
    
    ?>


总结: 这个笔记3的例子最终可以笼统认为:如果你`bindTo`的参数1是参数2的实例.两个参数所指的类同一个的.那么就可以认为`$this`写在了类内部.如果不统一,关联类作用域 和`$this` ,不是指同一个类.那么就可以认为$this不是写在类里面.

 Clear the stack trace: 清屏操作
	 
	 //下面这个例子,具体不太清楚用意,但大概意思还是指的是类作用域的造成的区别.
    <?PHP
    
    use Exception;     //这里新版php不需要在手动use , 你手动引入反而会报错The use statement with non-compound name 'Exception' has no effect  你的这个use使用了非复合(完整)名称,不会有影响,实际上因为这个类是内置类,压根不需用use引入就可以直接用了
    use ReflectionException; //这里也是一样,不需要引入
    
    $c = function()
    {
        $this->trace = [];
    };
    
    $c = $c->bindTo($R = new ReflectionException, Exception::class); //这里新版php已经不能正常运行了,因为参数2变成了不允许内置类作为参数传递
    
    $c();
    
    try
    {
      throw $R;
    } catch(ReflectionException $R) {
        var_dump($R->getTrace());
    }
    /*
    
    array(0) {
    
    }
    
    */
    
    ?>  
    

 ---------
 笔记4
 
Nezar Fadle 19-Jan-2015 10:40 


 We can use the concept of bindTo to write a very small Template Engine:
我们可以使用bindTo的思想去写一个非常小的模版引擎:

 **index.php**

    <?php
	//文章类
    class Article{
        private $title = "This is an article";
    }
    //邮报类
    class Post{
        private $title = "This is a post";
    }
    //模板类
    class Template{
		 //渲染方法 ,参数1:内容类, 参数2:模版文件
        function render($context, $tpl){
            $closure = function($tpl){
                ob_start(); //打开输出缓冲,也就是不会打印内容到浏览器
                include $tpl;
                return ob_end_flush();//冲刷出（送出）输出缓冲区内容并关闭缓冲区
            };
            $closure = $closure->bindTo($context, $context);
            $closure($tpl);
    
        }
    }
    
    $art = new Article();
    
    $post = new Post();
    
    $template = new Template();
    
    $template->render($art, 'tpl.php');
    
    $template->render($post, 'tpl.php');
    ?>



 **tpl.php**

     <h1><?php echo $this->title;?></h1> 


解释: 简单来说就是一个内容类和一个模板文件融合(include )起来,内容类里面的属性和模板文件要用到的属性要对应好,调用闭包的时候,就会让内容类里面的属性按照模板的要求显示出来,输出时,先缓存起来,全部编译完成了,再一次性输出全部内容.

 ----------
 笔记5
tatarynowicz at gmail dot com 14-Feb-2013 09:30 


 You can do pretty Javascript-like things with objects using closure binding:
你可以通过使用闭包绑定,做到一些很像JS里面的事情

    <?php
    
    trait DynamicDefinition {
    //在对象中调用一个不可访问方法时，__call魔术方法会被调用。
        public function __call($name, $args) {
            if (is_callable($this->$name)) {
                return call_user_func($this->$name, $args);
            }else {
                throw new \RuntimeException("Method {$name} does not exist");
            }
        }
	    //在给不可访问（protected 或 private）或不存在的属性赋值时,__set魔术方法会被调用。
        public function __set($name, $value) {
            $this->$name = is_callable($value)?$value->bindTo($this, $this): $value;
            //如果传进来的值是一个可以调用的,就创建一个属性,赋值为一个闭包,并且这个闭包绑定了当前的$this,和类作用域,这样就等同于这个闭包是定义在类内部的.不会受到访问权限限制.
        }
    }
    
    class Foo {
        use DynamicDefinition;
        private $privateValue = 'I am private';
    
    }
    $foo = new Foo;
    //这里会触发__set方法,$name是bar,$value就是后面的匿名函数
    $foo->bar = function() {
        return $this->privateValue;
    };
    
    // prints 'I am private'
    print $foo->bar(); //这里会触发__call方法,然后找bar这个属性,然后判断这个属性是可以调用的,然后进行了调用.
    ?>  

 ----------
 笔记6
safakozpinar at gmail dot com 09-Mar-2012 01:35 


 Private/protected members are accessible if you set the "newscope" argument (as the manual says).
私有/受保护的成员能够被连接, 如果你设置新的作用域,就像手册说的那样.


    <?php
    
    $fn = function(){
        return ++$this->foo; // increase the value
    };
    
    class Bar{
        private $foo = 1; // initial value
    }
    
    $bar = new Bar();
    $fn1 = $fn->bindTo($bar, 'Bar'); // specify class name+-
    $fn2 = $fn->bindTo($bar,  $bar); // or object //参数2可以是类名,或者类的实例
    
    echo $fn1(); // 2
    
    echo $fn2(); // 3  
    
	//注意这里参数1都是$bar,也就造成了$fn1和$fn2里面的$this都是指向同一个$bar实例(闭包内部的$this是引用传递的),所以造成了结果可以累加.
	//如果这个例子不想结果累加,就需要克隆复制$bar,不能是简单的$bar2= $bar ,这种也只是引用传递.需要使用clone克隆复制
	$bar3 = clone  $bar; //此时克隆了$foo的值是3,
	$fn3 = $fn->bindTo($bar3, $bar3);

	echo  $fn1(); // 4

	echo  $fn2(); // 5

	echo  $fn3(); // 4 .上面已经累加到5 了,但因为bar3是克隆复制,与之前bar的已经没有联系了,所以bar3还是从3开始+1

----------
笔记7
 
anthony bishopric 03-Mar-2012 01:00 


 Closures can rebind their $this variable, but private/protected methods and functions of $this are not accessible to the closures. 
 闭包可以重绑`$this`,但私有/受保护的成员不能访问.

    <?php
    $fn = function(){
        return $this->foo;
    };
    
    class Bar{
        private $foo = 3;
    }
    
    $bar = new Bar();
    $fn = $fn->bindTo($bar);
    echo $fn(); // Fatal error: Cannot access private property Bar::$foo  
	//这里例子根本就没点到他说的重点.
	//我们首先要知道,不是定义在类内部的匿名函数,天生就是没有$this 和类作用域的. 定义在类内部的,则天生就有这两项.
	//就这个例子,$fn天生就没有$this
	//如果想访问私有成员,那么除了绑$this,也必须同时给予同一个类的类作用域,否则$this只能访问公开成员.
	$fn = $fn->bindTo($bar,$bar);
	echo $fn(); //输出 3
 ----------
 笔记8
amica at php-resource dot de 18-Dec-2011 04:23 


 With rebindable `$this` at hand it's possible to do evil stuff:
拥有可以重绑的`$this`在手,你可以做一些坏事:

    <?php
        class A {
            private $a = 12;
            private function getA () {
                return $this->a;
            }
        }
    
        class B {
            private $b = 34;
            private function getB () {
                return $this->b;
            }
        }
    
        $a = new A();
        $b = new B();
        
        $c = function () {
            if (property_exists($this, "a") && method_exists($this, "getA")) {
                $this->a++;
                return $this->getA();
    
            }
    
            if (property_exists($this, "b") && method_exists($this, "getB")) {
                $this->b++;
                return $this->getB();
            }
        };
    
        $ca = $c->bindTo($a, $a);
    
        $cb = $c->bindTo($b, $b);
    
        echo $ca(), "\n"; // => 13
    
        echo $cb(), "\n"; // => 35
    
    ?>  

解释: 这个例子具体哪里比较坏不知道, 可能作者想表达的意思是容易弄混,容易调用错,得到预期外的结果?但种写法, 本身就源自两个不同的类,展示出不同的结果是很正常的啊.
