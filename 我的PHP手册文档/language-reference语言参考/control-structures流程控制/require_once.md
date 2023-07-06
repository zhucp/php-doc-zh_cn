## require_once

(PHP 4, PHP 5, PHP 7, PHP 8)

The `require_once` expression is identical to require except PHP will check if the file has already been included, and if so, not include (require) it again.
`require_once`  表达式和  [require]表达式完全相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。

----------
See the include_once documentation for information about the `_once` behaviour, and how it differs from its non `_once` siblings.
参见 [include_once] 的文档来理解 `_once` 的含义，并理解与没有 `_once` 时候有什么不同。

----------
### User Contributed Notes
笔记1

**bimal at sanjaal dot com** 04-Jun-2011 11:46

If your code is running on multiple servers with different environments (locations from where your scripts run) the following idea may be useful to you:  
如果您的代码运行在具有不同环境的多个服务器上（脚本运行的位置），则以下想法可能对您有用:
a. Do not give absolute path to include files on your server.   不要使用使用绝对路径
b. Dynamically calculate the full path (absolute path)  动态地计算完整路径(绝对路径)
  
Hints:  提示:
Use a combination of `dirname(__FILE__)` and subsequent calls to itself until you reach to the home of your '/index.php'. Then, attach this variable (that contains the path) to your included files.  
结合使用dirname() 和`__FILE__` , 直到你获得你的 '/index.php'文件(入口文件)所在的目录, 然后调用它自己.这样,固化这个变量给你的包含文件使用.
  
One of my typical example is:  我的其中一种典型例子:
  

    <?php  
	    define('__ROOT__', dirname(dirname(__FILE__)));  //调用两次dirname,获得上级目录路径
	    require_once(__ROOT__.'/config.php');  //配置文件在上一级目录中
    ?> 

 
  
instead of:  代替这个写法

    <?php require_once('/var/www/public_html/config.php'); ?>  //如果从Linux服务器换到windows服务器,那么这个路径是不存在的,这个写法就已经写死了代码只能运行在Linux上面,并且连目录名字也写死了.这个写法不可取.

  
After this, if you copy paste your codes to another servers, it will still run, without requiring any further re-configurations.  
这样之后,如果你复制粘帖你的代码到其他服务器,任然可以正常运行,无需进行任何重新配置.
  
  >这个笔记所讲的不要写死路径的做法,是所有项目都必须遵守的基本规则.不然你的项目将不具备可移植性.而只能在你的机器上运行.你也不能随意修改你的目录结构.所以任何路径,我们都不应该写死(使用绝对路径).而是应该使用相对路径或者动态计算出来的绝对路径
 
----------
笔记2

**bobray99 at softville dot com** 17-May-2022 05:52

Be careful when using include_once and require_once for files that return a value:  
请小心使用include_once和require_once,如果它们有返回值:
  
fiddle2.php  

    <?php  
    return "Some String";  

  
fiddle.php  

    <?php  
    $s = require_once('fiddle2.php');  
    echo "\n" . $s;  
    $s = require_once('fiddle2.php');  
    echo "\n" . $s;  

  

    /* output  
    Some String  
    1  
    */  

  
The second time require_once occurs, it returns 1 because the file has already been included.
第二次require_once结果是返回1,因为文件已经被包含过了.

----------
笔记3

**boda0128318 at gmail dot com** 08-Apr-2021 04:50

1 - "require" and "require_once" throw a fatal error if the file is not  
existing and stop the script execution  
1 - "require" 和 "require_once" 的文件如果不存在 ,脚本将会抛出一个致命错误,然后停止执行
  
2 - "include" and "include_once" throw a warning and the execution  
continues  
2 - "include" 和 "include_once" 抛出一个警告然后脚本会继续执行

3 - "require_once" and "include_once" as their names suggests ,  
they will not include the file if the file was already included with  
"require", "require_once", "include" or "include_once"  
3 - "require_once" 和 "include_once"正如它们的名字所言,
如果文件已经被"require", "require_once", "include" 或 "include_once"  包含过了,那么文件将不会被再次包含

try the following code:  尝试接下来的代码:
  
create a file called "index.php"   创建一个"index.php"文件
  

    <?php  
      
    require "first.php"; // this will include the file  这会成功包含这个文件
      
    include_once "first.php"; // this will not as it was included using "require"  这不会成功,因为它已经被"require" 包含过
      
    require_once "first.php"; // this will not as it was included using "require"  这也不会成功,因为它已经被"require" 包含过
      
    ?>  

  
and another file that is called "first.php" and write the following header  
另外一个文件叫"first.php",然后写入接下来的内容
    < h1>Hello every one</h1>  

i hope this will help you 我希望这些会帮到你


----------
笔记4

**powtac at gmx dot de** 02-Sep-2015 01:36

"require_once" and "require" are language constructs and not functions. Therefore they should be written without "()" brackets!
"require_once" 和 "require" 是语言结构,而不是函数.因此不应该写"()"括号!
>不写括号的写法同样也适用于"include"和"include_once"

----------
后续的负分比较不做翻译研究
