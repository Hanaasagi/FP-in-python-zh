具名函数和匿名函数
===================

显而易见的，在 Python 中创建 ``callables`` 的最明显的方法是具名函数和匿名函数。
原则上，两者之间的唯一区别是他们是否有 ``a.__qualname__`` 属性，即使都能够被绑定
到多个变量名上。在大多数场景中，lambda 表达式在 Python 中只用来作为回调函数或作为
简单的操作被内联至函数调用中。但就像我们先前所展示的那样，如果我们想要，控制流通常可以
被合并到单个匿名表达式中。举例说明如下::

    >>> def hello1(name):
    .....    print("Hello", name)
    .....
    >>> hello2 = lambda name: print("Hello", name)
    >>> hello1('David')
    Hello David
    >>> hello2('David')
    Hello David
    >>> hello1.__qualname__
    'hello1'
    >>> hello2.__qualname__
    '<lambda>'
    >>> hello3 = hello2
    # can bind func to other names
    >>> hello3.__qualname__
    '<lambda>'
    >>> hello3.__qualname__ = 'hello3'
    >>> hello3.__qualname__
    'hello3'

函数是有用的的原因之一是他们从词法上隔绝了状态，避免了命令空间污染。这是一种有限制的不可变形式，
虽然你在函数中什么都没做，依然绑定了函数外部的状态变量(只读)。当然，这种保证也是非常有限的，因为
`global` 和 `nonlocal` 语句都允许将状态变量“泄漏”给一个函数。此外，许多数据类型本身是可变的，
因此如果它们被传递到函数中可能会改变其值。此外，I/O 也可以改变状态，从而改变函数的结果(通过更改它
所读取的文件或数据库中的内容)

尽管存在上述所有的注意事项和限制，但想要专注于函数式编程风格的程序员可以有意地将许多函数写为纯函数
来进行数学和形式化的理解。In most cases, one only leaks state intentionally, and
creating a certain subset of all your functionality as pure functions allows
for cleaner code. They might perhaps be broken up by “pure” modules, or annotated
in the function names or docstrings.
