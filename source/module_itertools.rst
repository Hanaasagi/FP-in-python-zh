``itertools`` 模块
=====================
``itertools`` 模块是一个非常强大并且精心设计的用于 iterator algebra 的函数的集合。换句话说，
这些允许你使用高级的方式去组合迭代器，而无需去具体实例化任何比目前所需要的更多的东西。除了模块本身
的基本函数外，\ `模块文档 <https://docs.python.org/3.5/library/itertools.html>`_
provides a number of short, but easy to get subtly wrong, recipes for additional
functions that each utilize two or three of the basic functions in combination.
前言中提到的第三方模块 ``more_itertools`` 提供了额外的函数，同样旨在避免常见的陷阱和边缘情况。

使用 ``itertools`` 的基本目的是避免在需要之前进行计算，避免大型集合实例化所需要的内存，避免在
真正需求之前可能会有较慢的I / O，等等。迭代器是惰性序列而不是已经实现的集合，并且当使用 ``itertools``
和其他的函数相组合时，它们依旧保持特性。

这是结合几件事情的一个简单的例子。 我们可以简单地创建一个单一的惰性迭代器来生成当前的数字和这个总和：而不是有状态的斐波纳契（Fibonacci）类让我们保持一个运行的和，
