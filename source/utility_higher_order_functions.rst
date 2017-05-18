实用高阶函数
==============
*utility function 译者之前翻译成了工具函数，但本章节标题 utility higher-order functions
觉得这种翻译不太适用，所以又翻成了实用*

一个方便的工具是 ``compose()``。这是一个接受一序列函数并且返回一个函数，它接收一系列函数，
并返回一个函数。这个函数能将刚才传入的参数函数依次应用于数据参数::

    def compose(*funcs):
        """Return a new function s.t.
        compose(f,g,...)(x) == f(g(...(x)))"""
        def inner(data, funcs=funcs):
            result = data
            for f in reversed(funcs):
                result = f(result)
            return result
        return inner

    # >>> times2 = lambda x: x*2
    # >>> minus3 = lambda x: x-3
    # >>> mod6 = lambda x: x%6
    # >>> f = compose(mod6, times2, minus3)
    # >>> all(f(i)==((i-3)*2)%6 for i in range(1000000))
    # True


对于这些集中于一行的数学运算(times2，minus3等)，我们可以简单地写下对应原始的数学表达式；
但如果是涉及分支，流控制，复杂逻辑的复合计算，这不是真的。

内建函数 ``all()`` 和 ``any()`` 在判断 iterable 的元素是否对于谓词成立上很有用；但也能通过
组合来判断元素是否对于谓词集合成立，我们可以向下面这样实现::

    all_pred = lambda item, *tests: all(p(item) for p in tests)
    any_pred = lambda item, *tests: any(p(item) for p in tests)

为了展示用法，我们来制造些谓词::

    >>> is_lt100 = partial(operator.ge, 100)    # less than 100?
    >>> is_gt10 = partial(operator.le, 10)      # greater than 10?
    >>> from nums import is_prime               # implemented elsewhere
    >>> all_pred(71, is_lt100, is_gt10, is_prime)
    True
    >>> predicates = (is_lt100, is_gt10, is_prime)
    >>> all_pred(107, *predicates)
    False

``toolz`` 库有一个更通用的版本叫做 ``juxt()``，它创建一个使用相同参数调用多个函数并返回一个
结果元组的函数。如下所示::

    >>> from toolz.functoolz import juxt
    >>> juxt([is_lt100, is_gt10, is_prime])(71)
    (True, True, True)
    >>> all(juxt([is_lt100, is_gt10, is_prime])(71))
    True
    >>> juxt([is_lt100, is_gt10, is_prime])(107)
    (False, True, True)

上面用的实用高阶函数只是来说明可组合性的例子。看一下有关函数式编程的更详细的文本，例如，阅读
\ `Haskell prelude <https://hackage.haskell.org/package/base-4.8.0.0/docs/Prelude.html>`_
来获得有关实用高阶函数的许多概念。
