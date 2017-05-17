惰性求值
===========

Python 的一个强大功能是它的迭代器协议(我们很快便会了解到)。这个功能和函数式编程有这松散的联系，
因为 Python 并不像 Haskell 这类语言提供了惰性数据结构(lazy data structures)。然而，
使用迭代器协议和 Python 的一些内置或标准库中的可迭代类型可以实现与惰性数据结构相同的效果。

让我们稍微详细地解释这方面的对比。像 Haskell 这样本质是惰性求值的语言，我们可以像下面这样来定义
一个包含所有素数的列表::

    -- Define a list of ALL the prime numbers
    primes = sieve [2 ..]
      where sieve (p:xs) = p : sieve [x | x <- xs, (x `rem` p)/=0]

这篇文章不是来教 Haskell 的，但是你可以发现一个推导式，实际上这正是 Python 引入的推导式的原型。
这里还涉及到深度递归，这在 Python 中不会奏效。

除了语法差异，甚至是进行非确定深度递归的能力外，有个显著的差异便是 Haskell 版本的素数实际上是一个
(无限的)序列，而不仅仅是能够顺序生成元素的对象(Callables 章节提到过)。特别地，您可以根据下标获取
Haskell 无限列表中的任意元素(译者注 ``primes !! n``)，根据列表自身语法结构的需要，中间值会在
内部生成。

提醒一下，你可以将这套照搬进 Python 中，只是不是语言自身的固有语法，需要更多的手动构造。使用先前
讨论过的 ``get_primes()`` 生成器函数，我们可以编写自己的容器来模拟相同的事情，例如::

    from collections.abc import Sequence
    class ExpandingSequence(Sequence):
        def __init__(self, it):
            self.it = it
            self._cache = []
        def __getitem__(self, index):
            while len(self._cache) <= index:
                self._cache.append(next(self.it))
            return self._cache[index]
        def __len__(self):
            return len(self._cache)

新的容器同时具有惰性和可索引的特性::

    >>> primes = ExpandingSequence(get_primes())
    >>> for _, p in zip(range(10), primes):
    ....    print(p, end=" ")
    ....
    2 3 5 7 11 13 17 19 23 29
    >>> primes[10]
    31
    >>> primes[5]
    13
    >>> len(primes)
    11
    >>> primes[100]
    547
    >>> len(primes)
    101

当然，我们可能还会向其中添加一些其他功能，因为惰性数据结构不是 Python 自有的。也许我们想对这个特殊
的序列进行切片操作。也许我们希望在打印时更好地表示对象。也许我们应该把长度报告为 ``inf``，如果我们
能以某种方式证明它是无限的。所有这一切都是可能的，但它需要为每一种行为添加一些代码，而不是使用 Python
的默认行为。

.. toctree::
    :caption: 本章目录:

    the_iterator_protocol
    module_itertools
