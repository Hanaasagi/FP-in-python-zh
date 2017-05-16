闭包和可调用实例
=================

计算机科学中有一句名言:"class is data with operations attached while a closure is
operations with data attached"(类是附带操作的数据，闭包是附带数据的操作)。在某种意义上说，
他们完成了相同的事情，将逻辑和数据放在同一个对象中。但毫无疑问有着不同，类强调可变和能重新绑定的状态，
闭包强调不可变和纯函数。至少在 Python 中，这个分界的两侧没有一个是绝对的，使用哪种要看自己的态度。

让我们构建一个示例，来展现这两种风格::

    # A class that creates callable adder instances
    class Adder(object):
        def __init__(self, n):
            self.n = n
        def __call__(self, m):
            return self.n + m
    add5_i = Adder(5)   # "instance" or "imperative"

我们构造了一个能够被调用的实例，并且会将传入的参数与 5 相加。似乎已经够简单了，再来试试闭包版本::

    def make_adder(n):
        def adder(m):
            return m + n
        return adder
    add5_f = make_adder(5)   # "functional"

到目前为止，它们看起来相同，但实例中的可变状态提供了一个 attractive nuisance::

    >>>add5_i(10)
    15
    >>>add5_f(10)        # only argument affects result
    15
    >>>add5_i.n = 10     # state is readily changeable
    >>>add5_i(10)        # result is dependent on prior flow
    20

``Adder()``类和 ``make_adder()`` 所创建的 "adder" 的行为通常直到运行时才会确定。但一旦对象
(``add5_i``、``add5_f``)被创建，闭包就能显现出纯函数的特点，然而类实例仍然依赖着状态。人们可能
会通过互相约定"不要改变状态"来解决这个问题，确实这是可能的(如果没有其他不了解的人导入并使用了你的
代码)，但是避免滥用编程风格是更好的习惯。

Python 闭包中的变量绑定有一点问题。它通过 name 而不是 value 来，这可能会引起混淆，但也有一个
简单的解决方案。例如，制造几个相关的闭包封装不同的数据::

    # almost surely not the behavior we intended!
    >>> adders = []
    >>> for n in range(5):
            adders.append(lambda m: m+n)
    >>> [adder(10) for adder in adders]
    [14, 14, 14, 14, 14]
    >>> n = 10
    >>> [adder(10) for adder in adders]
    [20, 20, 20, 20, 20]

幸运的是，一个小小的改变便能使行为符合预期::

    >>> adders = []
    >>> for n in range(5):
    ....     adders.append(lambda m, n=n: m+n)
    ....
    >>> [adder(10) for adder in adders]
    [10, 11, 12, 13, 14]
    >>> n = 10
    >>> [adder(10) for adder in adders]
    [10, 11, 12, 13, 14]
    >>> add4 = adders[4]
    >>> add4(10, 100)   # Can override the bound value
    110

Notice that using the keyword argument scope-binding trick allows
you to change the closed-over value; but this poses much less of a
danger for confusion than in the class instance. The overriding
value for the named variable must be passed explictly in the call
itself, not rebound somewhere remote in the program flow. Yes, the
name add4 is no longer accurately descriptive for “add any two
numbers,” but at least the change in result is syntactically local.
