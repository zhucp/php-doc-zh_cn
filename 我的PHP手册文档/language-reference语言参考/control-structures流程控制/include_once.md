## include_once

(PHP 4, PHP 5, PHP 7, PHP 8)

The `include_once` statement includes and evaluates the specified file during the execution of the script. This is a behavior similar to the include statement, with the only difference being that if the code from a file has already been included, it will not be included again, and include_once returns **`true`**. As the name suggests, the file will be included just once.
`include_once` 语句在脚本执行期间包含并运行指定文件。此行为和 [include]语句类似，唯一区别是如果该文件中已经被包含过，则不会再次包含，且 include_once 会返回 **`true`**。 顾名思义，require_once，文件仅仅包含（require）一次。

`include_once` may be used in cases where the same file might be included and evaluated more than once during a particular execution of a script, so in this case it may help avoid problems such as function redefinitions, variable value reassignments, etc.
`include_once` 可以用于在脚本执行期间同一个文件有可能被包含超过一次的情况下，想确保它只被包含一次以避免函数重定义，变量重新赋值等问题。

See the include documentation for information about how this function works.
更多信息参见 [include]文档。
### User Contributed Notes
没有用户笔记
