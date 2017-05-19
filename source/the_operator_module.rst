``operator`` 模块
======================

像先前几个例子所展示的那样，每个使用 Python 中缀和前缀运算符来完成的操作都在 ``operator``模块
存在对应的函数。对于希望向高阶函数传入相当于某些语法操作的函数作为参数的地方，使用 ``operator``
模块的函数更快，并且看起来比相应的匿名函数更优雅，例如::

    # Compare ad hoc lambda with `operator` function
    sum1 = reduce(lambda a, b: a+b, iterable, 0)
    sum2 = reduce(operator.add, iterable, 0)
    sum3 = sum(iterable)   # The actual Pythonic way
