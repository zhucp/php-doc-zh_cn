## Closure 类

(PHP 5 >= 5.3.0, PHP 7, PHP 8)
简介 

---------
用于代表(实现) 匿名函数的类. 
匿名函数会产生(PHP内部自动把匿名函数转成)这种类型的对象。这个类带有一些方法允许在匿名函数创建后对其进行更多的控制 (内置了一些方法)。
除了此处列出的方法，还有一个 `__invoke` 方法。这是为了与其他实现了 `__invoke()`魔术方法的对象保持一致性，但调用匿名函数的过程与它无关(不是通过触发__invoke从而调用执行匿名函数的,但是如果你直接调用闭包的__invoke(),也会执行匿名函数)。 

---------
类摘要 

    final class Closure {
    /* 方法 */
    private __construct()
    public static bind(Closure $closure, ?object $newThis, object|string|null $newScope = "static"): ?Closure
    public bindTo(?object $newThis, object|string|null $newScope = "static"): ?Closure
    public call(object $newThis, mixed ...$args): mixed
    public static fromCallable(callable $callback): Closure
    }

Table of Contents 
•Closure::__construct — 用于禁止实例化的构造函数
•Closure::bind — 用特定的绑定对象和类作用域复制闭包。
•Closure::bindTo — 用特定的绑定对象和类作用域复制闭包。
•Closure::call — 绑定并调用闭包
•Closure::fromCallable — 将 callable 转换为闭包

---------

User Contributed Notes :用户笔记


笔记1
info at ensostudio dot ru 21-Mar-2022 05:41 

 compare closures:比较两个匿名函数是否是指向同一个闭包类

    <?php 
    
    (string) new ReflectionFunction($fn) === (string) new ReflectionFunction($fn2)
    
    ?>  

ReflectionFunction 类报告有关函数的信息。可以用来得到函数的各种信息.
这里的逻辑:把`$fn` 通过ReflectionFunction转为一条字符串,然后和`$fn2`的字符串结果做比较,看是否完全一样.
我们新增一个例子: 

    function createGreeter($who) {
              return function() use ($who) {
                  echo "Hello $who";
              };
	}
    $greeter = createGreeter("World");
    $greeter2 = createGreeter("World2");
    var_dump((string) new ReflectionFunction($greeter));
    var_dump((string) new ReflectionFunction($greeter2));

打印出来的结果:

    string(203) "Closure [ <user> function {closure} ] {
      @@ C:\...这里是绝对路径,我省略了...\26.php 3 - 5
    
      - Bound Variables [1] {
          Variable #0 [ $who ]
      }
    }
    "
    string(203) "Closure [ <user> function {closure} ] {
      @@ C:\...这里是绝对路径,我省略了...\26.php 3 - 5
    
      - Bound Variables [1] {
          Variable #0 [ $who ]
      }
    }
    "

可以看到完全一样的字符串!记录了函数的结构信息,具体的`$who`是什么值,这个属性的值不在比较范围,
如果我们原封不动地复制匿名函数的定义,创建另外一个:

    $greeter2 =  function() use ($who) {
                      echo "Hello $who";
                  };

结果:

    string(203) "Closure [ <user> function {closure} ] {
      @@ C:\...这里是绝对路径,我省略了...\26.php 3 - 5
    
      - Bound Variables [1] {
          Variable #0 [ $who ]
      }
    }
    "
    string(205) "Closure [ <user> function {closure} ] {
      @@ C:\...这里是绝对路径,我省略了...\26.php 11 - 13
    
      - Bound Variables [1] {
          Variable #0 [ $who ]
      }
    }
    "

我们看到展示出来的机构是一样的,但具体到所在的文件和行数不一样了.这说明这两个函数不是同一个定义来源.只是样子长得一模一样.
通过这个ReflectionFunction类,我们比较得到函数的各种信息,也可以进行两个函数的比较,无论是普通函数还是匿名函数;
 
 ----------
 笔记2
joe dot scylla at gmail dot com 08-Apr-2016 08:46 


 Small little trick. You can use a closures in itself via reference.
小技巧。您可以通过引用,在闭包内部调用闭包自己。

 Example to delete a directory with all subdirectories and files:
删除目录和它的所有子目录和文件的例子:

    <?php
    $deleteDirectory = null;
    $deleteDirectory = function($path) use (&$deleteDirectory) {
        $resource = opendir($path); //打开一个目录句柄，可用于之后的 closedir()，readdir() 和 rewinddir() 调用中。
        while (($item = readdir($resource)) !== false) { //返回目录中下一个文件的文件名。文件名根据在文件系统中的排序返回。成功则返回文件名 或者在失败时返回 false
            if ($item !== "." && $item !== "..") {//一个点是当前目录,两个点是上一级目录,这两个要排除掉
                if (is_dir($path . "/" . $item)) {//如果是还是目录,就递归
                    $deleteDirectory($path . "/" . $item);
                } else {//不是目录就删除
                    unlink($path . "/" . $item);
                }
            }
        }
        closedir($resource);//关闭由 dir_handle 指定的目录流。流必须之前被 opendir() 所打开。
        rmdir($path);//最后删除最顶级的目录
    };
    $deleteDirectory("path/to/directoy");
    ?> 
---------
笔记3
 
luk4z_7 at hotmail dot com 14-Jun-2015 05:42 

 Scope 作用域
 A closure encapsulates its scope, meaning that it has no access to the scope in which it is defined or executed. It is, however, possible to inherit variables from the parent scope (where the closure is defined) into the closure with the use keyword:
闭包封装了其作用域，这意味着它无法访问定义或执行它的作用域。但是，可以使用use关键字将变量从父作用域（定义闭包的地方）继承到闭包中：

     function createGreeter($who) {
                   return function() use ($who) { //对这个闭包,定义它的地方就是这个函数内部,父作用域就是这个函数内部,任何可以在这个函数内部访问的变量,都可以用use 传递到闭包内部.
                       echo "Hello $who";
                   };
     }
    
     $greeter = createGreeter("World");
     $greeter(); // Hello World

 This inherits the variables by-value, that is, a copy is made available inside the closure using its original name.
这按值继承变量(值传递)，也就是说，使用其原始名称在闭包内部提供(新建)一个副本。闭包内的变量和闭包外的没有联系,除非你在use中指明用引用传递(例如:  `&$a`);
 
 ----------
 笔记4
chuck at bajax dot us 09-Jun-2015 04:16 

 This caused me some confusion a while back when I was still learning what closures were and how to use them, but what is referred to as a closure in PHP isn't the same thing as what they call closures in other languages (E.G. JavaScript).
当我还在学习闭包是什么以及如何使用它们时，这给我带来了一些困惑，但在PHP中被称为闭包的东西与他们在其他语言中所称的闭包（例如JavaScript）不同。

 In JavaScript, a closure can be thought of as a scope, when you define a function, it silently inherits the scope it's defined in, which is called its closure, and it retains that no matter where it's used.  It's possible for multiple functions to share the same closure, and they can have access to multiple closures as long as they are within their accessible scope.
在JavaScript中，闭包可以被认为是一个范围，当你定义一个函数时，它会默默地继承它在其中定义的范围，这被称为它的闭包，无论在哪里使用，它都会保留它。多个函数可以共享同一个闭包，并且只要它们在其可访问范围内，它们就可以访问多个闭包。

 In PHP,  a closure is a callable class, to which you've bound your parameters manually.
在PHP中，闭包是一个可调用的类，您可以手动将参数绑定到该类。

 It's a slight distinction but one I feel bears mentioning. 
这是一个细微的区别，但我觉得值得一提。
