装饰器
==========

虽然设计上容易被忘记，但 Python 中最常见的高阶函数用法可能就是装饰器。装饰器只是一种将一个函数作为
参数的语法糖，如果编写正确，则会返回一种新函数，它以某种方式增强原始函数(或方法或类)。提醒读者，
这两个代码片段定义的 ``some_func`` 和 ``other_func`` 是等价的::

    @enhanced
    def some_func(*args):
        pass
    def other_func(*args):
        pass
    other_func = enhanced(other_func)

使用装饰器语法，当然，高阶函数必须用于函数定义时。为了达到预定的目的，这通常是最好的应用。但原则上
同样的装饰器函数可以在程序的其他地方使用，例如以更动态的方式(e.g., mapping a decorator
function across a runtime-generated collection of other functions)。然而，这是一个
不常用的例子。

装饰器被使用在标准库和常见的第三方库中的许多场合里。在某些方面，它们与以前称为"面向切面的程序设计"
的想法相结合。例如，装饰器函数 ``asyncio.coroutine`` 用于将函数标记为协程。在 ``functools``
中，三个重要的装饰器函数是 ``functools.lru_cache``， ``functools.total_ordering`` 和
``functools.wraps``。``functools.lru_cache`` 能"记住""一个函数(缓存传入的参数，返回存储
的值，而不是重新进行计算或者 I/O)。``functools.total_ordering`` 能让自定义类实现比较运算
更加容易。``functools.wraps`` 使得更容易编写新的装饰器。所有这些都是重要和有价值的目的，但是
他们是本着使 Python 编程更简单的精神，而不是着眼于本章重点关注的可组合的高阶函数。

装饰器通常在你想要 poke into the guts of a function 时比你仅将它视为函数流或者组合中的可拔插
组件时更有用，它经常用来标记特定函数的目的或功能。

这篇文章只是大致讲解了 Python 函数式编程的相关技术和 some suggestions as to the advantages
one often finds in aspiring in that direction. 编写没有副作用的函数可以避免调试错误时遇到
的一大类困难，而且编写更容易理解和进行可靠测试的小型功能单元可以避免更多的错误。

A rich literature on functional programming as a general technique often in
particular languages which are not Python—is available and well respected.
Studying one of many such classic books, some published by O’Reilly (including
very nice video training on functional programming in Python), can give readers
further insight into the nitty-gritty of functional programming techniques.
Almost everything one might do in a more purely functional language can be done
with very little adjustment in Python as well.
