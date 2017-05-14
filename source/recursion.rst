递归
========
习惯于函数式编程的程序员经常使用通过递归来表达循环。这样做，我们可以避免在算法内
改变任何变量或数据结构的状态，and more importantly get more at the “what”
than the “how” of a computation.然而在考虑使用递归风格时，我们应该区分这两种
情形：1)递归仅仅是迭代的另一个名称；2)递归将问题划分为处理方式类似的较小问题

有两个原因使我们应当在这里提及这个区别：

1)使用递归虽然可能是一种高效的方法来遍历序列中的每一个元素，但真的不 "Pythonic"。这是
其他语言的风格，比如 Lisp，在 Python 中显得矫揉造作。

2)Python 在递归中相对较慢，还有着栈深度限制。虽然你可以通过 ``sys.setrecursionlimit()``
来使其超过默认值 1000 (译者注 2.x 为 1000; 3.x 为 2000)。但这样做，可能会是个错误的决定。
Python 没有尾调用消除，在某些语言中使用尾调用消除可以高效地进行深度递归计算。让我们看一个递归
是迭代的另一种形式的例子，可能会有些无聊::

    def running_sum(numbers, start=0):
        if len(numbers) == 0:
            print()
            return
        total = numbers[0] + start
        print(total, end=" ")
        running_sum(numbers[1:], total)

然而，很少有人推荐这种方法；相比之下，简单重复地修改状态变量的迭代可读性会更高，
