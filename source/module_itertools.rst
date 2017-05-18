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

这里有组合的一个简单的例子。比起有状态的 ``Fibonacci`` 类来维持累加和，让我们创建一个惰性迭代器
来生成当前的值和总和::

    >>> def fibonacci():
    ...     a, b = 1, 1
    ...     while True:
    ...         yield a
    ...         a, b = b, a+b
    ...
    >>> from itertools import tee, accumulate
    >>> s, t = tee(fibonacci())
    >>> pairs = zip(t, accumulate(s))
    >>> for _, (fib, total) in zip(range(7), pairs):
    ...     print(fib, total)
    ...
    1 1
    1 2
    2 4
    3 7
    5 12
    8 20
    13 33

搞清楚如何正确，最佳地使用 ``itertools`` 模块中的函数通常需要经过缜密的思考，但一旦成功，可以获得
显著的能力来处理那些无法用具体集合表现的大型甚至无穷的迭代器。

``itertools`` 模块的文档包含了其函数的详细信息以及用于组合它们的许多简短样例。这篇文章没有地方去
再次重复这些描述，所以仅展示其中的几个。注意实际上，``zip()``， ``map()``， ``filter()`` 和
``range()``(某种意义上只是一个终止的 ``itertools.count()``)应当在 ``itertools`` 模块，
如果他们不是 builtins。所有的这些函数都会惰性生成序列项(大部分基于现有的迭代)，而不创建具体的序列。
``all()``, ``any()``, ``sum()``, ``min()``, ``max()`` 和 ``functools.reduce()``
也能用在 iterables 上，但他们需要将迭代器完全迭代，无法再保留其惰性了。函数 ``itertools.product()``
可能在不应当放在这个模块中，因为它还创建具体的缓存序列，并且无法在无穷迭代器上操作。

**Chaining Iterables**

``itertools.chain()`` 和 ``itertools.chain.from_iterable()`` 函数能够将多个迭代器进行
组合。Built-in ``zip()`` and ``itertools.zip_longest()`` also do this, of course,
but in manners that allow incremental advancement through the iterables. 这样做的
结果是，连接无穷 iterables 在语法和语义上是有效的，没有实际的程序能够迭代完早期的可迭代。例如::

    from itertools import chain, count
    thrice_to_inf = chain(count(), count(), count())

概念上 ``thrice_to_inf`` 会三次计算至无穷，但实际上一次就足够了(in practice once would
always be enough)。然而，对于很长的 iterables (非无穷)，使用 chain 是非常有用和简约的::

    def from_logs(fnames):
        yield from (open(file) for file in fnames)
    lines = chain.from_iterable(from_logs(
                  ['huge.log', 'gigantic.log']))

请注意，在给出的示例中，我们甚至不需要传递具体的文件列表——这个列表可以是指定 API 的惰性迭代器。
除了使用 ``itertools`` 进行连接之外，我们还应该一同提及 ``collections.ChainMap()``。字典
类型(通常任何的 ``collections.abc.Mapping`` 类型)是可迭代的。就像我们想要去连接多个类似序列
的 iterables 一样，我们有时希望连接多个映射(mappings)，而不需要直接创建一个更大的具体映射。
``ChainMap()`` 方便，并且没有改变构造它的底层映射。
