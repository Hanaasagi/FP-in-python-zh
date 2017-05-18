高阶函数
===========

在上一章中，我们看到了建立在 ``itertools`` 模块的迭代器代数。在某些方面，高阶函数(通常缩写为
"HOFs")通过组合简单函数至新的函数来提供了类似的表达复杂概念的方式。一般来说，高阶函数是一个接受
一个或多个函数作为参数或者产生新函数作为结果的函数。这里提供了许多有趣的抽象。它们允许我们像使用
``itertools`` 那样类似的方式进行连接和组合高阶函数。

``functools`` 模块中包含一些有用的高阶函数，另一些则已经内建。通常认为，``map()``, ``filter()``,
和 ``functools.reduce()`` 是高阶函数的最基本构建块，大多数函数式编程语言将他们作为原语(偶尔
使用其他的名字)。像 map/filter/reduce 作为构建块般基础的概念是柯里化。在 Python 中，柯里化
被叫做 ``partial()``，它在 ``functools`` 模块中。这个函数，它接收一个函数以及零个或多个参数
然后进行预填充，返回一个较少参数的函数。调用时，就像参数已经传递过了一样。

内建函数 ``map()`` 和 ``filter()`` 等同于推导(特别是现在可以使用生成器推导)，而且大多数
Python 程序员都认为使用推导的版本更易于阅读。例如下面这些等效的组合::

    # Classic "FP-style"
    transformed = map(tranformation, iterator)
    # Comprehension
    transformed = (transformation(x) for x in iterator)

    # Classic "FP-style"
    filtered = filter(predicate, iterator)
    # Comprehension
    filtered = (x for x in iterator if predicate(x))

``functools.reduce()`` 函数非常通用，非常强大，并且能非常巧妙地发挥其全部功能。它从一个 iterable
中依次去除相连元素，并以某种方式进行组合。``reduce()`` 的最常见的用例涵盖了内建的 ``sum()``，
下面一种更简洁的写法::

    from functools import reduce
    total = reduce(operator.add, it, 0)
    # total = sum(it)

``map()`` 和 ``filter()`` 也可以被认为是 ``reduce()`` 的特殊情况::

    >>> add5 = lambda n: n+5
    >>> reduce(lambda l, x: l+[add5(x)], range(10), [])
    [5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
    >>> # simpler: map(add5, range(10))
    >>> isOdd = lambda n: n%2
    >>> reduce(lambda l, x: l+[x] if isOdd(x) else l, range(10),
    [])
    [1,3, 5, 7, 9]
    >>> # simpler: filter(isOdd, range(10))

这些 ``reduce()`` 表达式是尴尬的，但它们也说明了该函数在其通用性方面的强大程度：任何需要从序列
取出连续元素来计算的操作都可以被表示为 reduction。

有几个常见的高阶函数没有被 Python 所包含，但很容易作为工具函数来创建(并且包含在许多第三方函数式
编程工具集合中)。不同的库或其他编程语言可能使用与上述工具函数所不同的名称，但类似的功能是广泛存在
(我选择的名称也是如此)。

.. toctree::
    utility_higher_order_functions
    the_operator_module
    the_functools_module
    decorators
