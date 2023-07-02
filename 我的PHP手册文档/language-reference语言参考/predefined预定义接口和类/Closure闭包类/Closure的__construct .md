## Closure::__construct

(PHP 5 >= 5.3.0, PHP 7, PHP 8)

Closure::__construct — 用于禁止实例化闭包类的构造函数

### 说明

private **Closure::__construct**()

这个方法仅用于禁止实例化一个 Closure 类的对象。这个类的对象的创建方法写在 匿名函数 页。也就是你想创建一个闭包类,只能通过创建匿名函数来实现.

### 参数

此函数没有参数。

### 参见

-   匿名函数

### User Contributed Notes

笔记1

    <?php
    $a = new Closure();

报错: Instantiation of class Closure is not allowed :不允许实例化Closure类
