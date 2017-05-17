迭代器协议
=============

在 Python 中创建一个迭代器(或者说一个惰性序列)的最简单的方法是定义一个生成器函数，如"Callables"
一章所讨论的那样。只需在函数体内需要生成值的位置(通常在循环中)使用 ``yield`` 语句即可。

或者最简单的方法是使用 builtin 或标准库提供许多可迭代对象，而不是自己从头编写一个对象。生成器函数
是定义一个返回迭代器的函数的语法糖。

许多对象含有用来返回迭代器的 ``.__iter__()`` 方法，这个方法通常被内建函数 ``iter()`` 或者循环这个
对象(e.g., for item in collection: ...)时调用。

迭代器是通过调用 ``iter(something)`` 所产生的对象，它自身也具有 ``.__iter__()`` 方法，这个
方法会返回它本身。迭代器具有的另一个方法是 ``.__next__()``。迭代器自身仍具有 ``.__iter__()``
方法的原因是为了让 ``iter()`` 幂等。That is, this identity should always hold
(or raise TypeError("object is not iterable") )::

    iter_seq = iter(sequence)
    iter(iter_seq) == iter_seq

上面讲的有点抽象，所以我们来看几个具体的例子::

    >>> lazy = open('06-laziness.md') # iterate over lines of file
    >>> '__iter__' in dir(lazy) and '__next__' in dir(lazy)
    True
    >>> plus1 = map(lambda x: x+1, range(10))
    >>> plus1                  # iterate over deferred computations
    <map at 0x103b002b0>
    >>> '__iter__' in dir(plus1) and '__next__' in dir(plus1)
    True
    >>> def to10():
    ...     for i in range(10):
    ...         yield i
    ...
    >>> '__iter__' in dir(to10)
    False
    >>> '__iter__' in dir(to10()) and '__next__' in dir(to10())
    True
    >>> l = [1,2,3]
    >>> '__iter__' in dir(l)
    True
    >>> '__next__' in dir(l)
    False
    >>> li = iter(l)          # iterate over concrete collection
    >>> li
    <list_iterator at 0x103b11278>
    >>> li == iter(li)
    True

在函数式编程中，或者为了追求可读性，使用生成器函数来编写自定义迭代器是再自然不过的事了。不过，我们还
可以创建遵循协议的类；通常它们也具有其他的行为(i.e., methods)，但大多数此类行为依赖于有状态和副作用。
例如::

    from collections.abc import Iterable
    class Fibonacci(Iterable):
        def __init__(self):
            self.a, self.b = 0, 1
            self.total = 0
        def __iter__(self):
            return self
        def __next__(self):
            self.a, self.b = self.b, self.a + self.b
            self.total += self.a
            return self.a
        def running_sum(self):
            return self.total

    # >>> fib = Fibonacci()
    # >>> fib.running_sum()
    # 0
    # >>> for _, i in zip(range(10), fib):
    # ...     print(i, end=" ")
    # ...
    # 1 1 2 3 5 8 13 21 34 55
    # >>> fib.running_sum()
    # 143
    # >>> next(fib)
    # 89

这个例子不值一提，但它展示了一个实现迭代器协议，并且还提供了额外的方法来返回实例状态的类。由于使用状态
是面向对象程序员经常做的，在函数式编程风格中，我们通常会避免这样的类。
