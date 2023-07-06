## require

(PHP 4, PHP 5, PHP 7, PHP 8)

`require` is identical to include except upon failure it will also produce a fatal **`E_COMPILE_ERROR`** level error. In other words, it will halt the script whereas include only emits a warning (**`E_WARNING`**) which allows the script to continue.
`require`  和  `include` 几乎完全一样，除了处理失败的方式不同之外。**require**  在出错时产生  **`E_COMPILE_ERROR`**  级别的错误。换句话说将导致脚本中止而   **include** 只产生警告（**`E_WARNING`**），脚本会继续运行。

----------
See the include documentation for how this works.
由于几乎都一样,所以具体用法请参见  include 文档了解详情。

----------
### User Contributed Notes   用户贡献的笔记
笔记1
**chris at chrisstockton dot org** 19-Jun-2007 05:06

Remember, when using require that it is a statement, not a function. It's not necessary to write:  
请记住,require 是一个声明,不是一个函数,所以没有必要这样写:

    <?php  
    require('somefile.php');  //无论加不加括号都可以
    ?>  

  
The following:  而是应该这样写

    <?php  
    require 'somefile.php';  
    ?>  

  
Is preferred, it will prevent your peers from giving you a hard time and a trivial conversation about what require really is.
更可取的是，这将防止你的同龄人给你带来困难，并就真正需要什么进行琐碎的交谈。
>更进一步来说,这个例子可以防止你的同事给你带来麻烦和一些琐碎的解释,有关于require到底是什么.

----------
笔记2

**Marcel Baur** 27-Jul-2021 09:08

If your included file returns a value, you can get it as a result from require(), i.e.  
 如果你使用包含一个有返回值的文件,那么你用请求也一样可以得到相同的结果,也就是: 
  
foo.php:  

    <?php  
    return "foo";  
    ?>  
      
    $bar = require("foo.php");  
    echo $bar; // equals to "foo"

这个行为和include是一样的

----------

剩余十几个笔记都是负分笔记,不做翻译研究
