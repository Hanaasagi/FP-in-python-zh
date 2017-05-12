（Avoiding）Flow Control
========================

在典型的 Python 命令式编程(包括将命令式代码存放于类和方法)中，
代码块通常包含一些循环 (``for`` or ``while``)，状态变量的赋值，
``dict``, ``list``, ``set``, 及各种各样来自于标准库或第三方包的
数据结构的修改，与分支操作(``if``/``elif``/``else``,
``try``/``except``/``finally``)。所有的这些都是自然的，下意识便能够理解。
然而那些经常出现的问题，恰恰来自于这些状态变量，可变数据结构所带来的副作用；
他们虽然在模拟物理世界的概念，却难以准确解释程序中指定点处的状态数据

一个解决方案是不要专注于构建数据集合，而是描述数据集合中包含了什么。
When one simply thinks, “Here’s some data, what do I need to do with it?”
rather than the mechanism of constructing the data, more direct
reasoning is often possible. The imperative flow control described in
the last paragraph is much more about the “how” than the “what”
and we can often shift the question.

.. toctree::

    encapsulation
    comprehensions
    recursion
    eliminating_loops
