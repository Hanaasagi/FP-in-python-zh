消除循环
==========
Just for fun，让我们看看如何从 Python 程序中删除所有循环。大多数时候，无论是对于可读性还是
性能来说，这是个坏主意，but it is worth looking at how simple it is to do
in a systematic fashion as background to contemplate those cases where it
is actually a good idea.(译者不知所云：但值得一看的是，在系统化背景下消除循环是一个
不错的主意去容易的思考一些场景)

如果我们在 ``for`` 循环内部调用一个函数，那么內建高阶函数 ``map()`` 可以给予我们帮助::

    for e in it:   # statement-based loop
        func(e)

下面的代码完全等价，除了没有重复重地新绑定变量 ``e``，因此没有状态::

    map(func, it)   # map()-based "loop"

类似的技术可用于利用函数式的方法来实现顺序程序流。大多数命令式程序都是由诸如"先做这，再做那个，
然后再做其他的"这样的语句所组成。如果这些操作全放到了函数中，那么 ``map()`` 就可以让我们这样做::

    # let f1, f2, f3 (etc) be functions that perform actions
    # an execution utility function
    do_it = lambda f, *args: f(*args)
    # map()-based action sequence
    map(do_it, [f1, f2, f3])

我们可以将函数序列和传递对应参数的迭代器进行组合::

    >>> hello = lambda first, last: print("Hello", first, last)
    >>> bye = lambda first, last: print("Bye", first, last)
    >>> _ = list(map(do_it, [hello, bye],
    >>>                     ['David','Jane'], ['Mertz','Doe']))
    Hello David Mertz
    Bye Jane Doe

当然，看这个例子，有人猜想结果会是将所有的参数传入每一个函数而不是从每一个列表取一个参数传入函数。
若不使用列表推导表达起来是比较困难的::

    >>> do_all_funcs = lambda fns, *args: [
                              list(map(fn, *args)) for fn in fns]
    >>> _ = do_all_funcs([hello, bye],
                         ['David','Jane'], ['Mertz','Doe'])
    Hello David Mertz
    Hello Jane Doe
    Bye David Mertz
    Bye Jane Doe

一般来说，我们的整个主程序原则上可以是一个含有用来完成程序的可迭代函数集的 ``map()`` 表达式。
转换虽然稍微复杂一些，但可以直接使用递归::

    # statement-based while loop
    while <cond>:
        <pre-suite>
        if <break_condition>:
            break
        else:
            <suite>

    # FP-style recursive while loop
    def while_block():
        <pre-suite>
        if <break_condition>:
            return 1
        else:
            <suite>
        return 0

    while_FP = lambda: (<cond> and while_block()) or while_FP()
    while_FP()

我们对于 ``while`` 的转换仍然需要一个 ``while_block()`` 函数，它可能不仅仅包含表达式，或许
还会有语句。我们可以使用上面提到的 ``map()`` 来进一步将组件转换成函数序列。如果我们这样做，
还可以返回一个的三元表达式。将这个进一步的函数式重构作为一个练习留给读者。它可能会比较丑陋，
并且不适合生产环境，但是很好玩。译者觉得可能是这样？

  def while_block():
      <map(<pre-suite>)>
      return 1 if <break_condition> else 0

  while_FP = lambda: (<cond> and (while_block() or <map(suite)>)) or while_FP()
  while_FP()

使用通常的测试手段，``<cond>`` 很难有利用价值，比如 ``while myvar==7``，因为循环体(按设计)
不能改变任何变量的值。一种添加更有用的条件的方法是让 ``while_block()`` 返回一个更有趣的值，
并比较返回值是否为终止条件。以下是一个删除语句的具体示例::

    # imperative version of "echo()"
    def echo_IMP():
        while 1:
            x = input("IMP -- ")
            if x == 'quit':
                break
            else:
                print(x)
    echo_IMP()

现在让我们来删除 ``while`` 循环::

    # FP version of "echo()"
    def identity_print(x):   # "identity with side-effect"
        print(x)
        return x

    echo_FP = lambda: identity_print(input("FP -- "))=='quit' or echo_FP()
    echo_FP()

我们已经完成的是，设法将一个涉及 I/0，循环和条件语句的小程序表达为纯粹的递归表达式(实际上，作为
一个可以传递到别处的函数对象)。我们仍然使用工具函数 ``identity_print()``，这个函数是完全通用的，
能够在稍后创建的每个函数式程序表达式中重用。请注意，包含 ``identity_print(x)`` 的任何表达式的
计算结果和简单地包含 ``x`` 相同; 它只是包含了额外的 I/O 操作。

**消除递归**

和上面所给出的阶乘示例相似，有时我们可以通过 ``functools.reduce()`` 或其他的 fold 操作(其他
folds 不在 Python 标准库中，但可以通过第三方库构建)来实现隐式的递归。递归通常只是将一些更简单的
结果与累积的中间结果相结合的方法，这正是 ``reduce()`` 内部所做的。关于 ``functools.reduce()``
的更多讨论在高阶函数的章节。
