递归
========
习惯于函数式编程的程序员经常使用递归来表达循环。这样做，我们可以避免在算法内改变任何变量或数据结构
的状态，更重要的是能够更多的了解 "What"。然而在考虑使用递归风格时，我们应该区分这两种情形：
1)递归仅仅是迭代的另一个名称；2)递归将问题划分为处理方式类似的较小问题

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

然而，很少有人推荐这种方法；相比之下，简单重复地修改状态变量的迭代可读性会更高，此外当 ``numbers``
长度大于 1000 时，这个函数会成为解释栈溢出的完美示例。但在其他情况下，递归风格，即使是顺序操作，
仍能更直观地表达算法，并且是以一种更容易理解的方式。以阶乘举例

递归版本::

    def factorialR(N):
        "Recursive factorial function"
        assert isinstance(N, int) and N >= 1
        return 1 if N <= 1 else N * factorialR(N-1)


迭代版本::

    def factorialI(N):
      "Iterative factorial function"
      assert isinstance(N, int) and N >= 1
      product = 1
      while N >= 1:
          product *= N
          N -= 1
      return product

虽然这个算法使用 ``product`` 变量就能够被容易地表达，但使用递归表示更倾向于算法的 "What"
而不是 "How"。迭代版本中反复地改变 ``product`` 和 ``N`` 的值，感觉就像是在记账，而不是
寻求计算的本质(但迭代版本大概更快，并且没有限制)。

多说一点，我所知道的 Python 中 ``factorial()`` 的最快版本是函数式编程风格的，并且很好地
表达了算法的 "What"，不过要熟悉一些高阶函数::

    from functools import reduce
    from operator import mul
    def factorialHOF(n):
        return reduce(mul, range(1, n+1), 1)

某些场景下，我们不得不使用递归。有时递归甚至是表达解决方案的唯一明显方式，例如当一个问题提供了
将自己分治的途径。分治换句话说就是能够对一个大集合的两半(或者，几个类似大小的块)进行相似的计算。
在这种情况下，递归深度不会太大，仅为 O(log N)， N 为序列的长度。举个例子，快速排序算法十分
优雅，不需要任何状态变量或循环，完全通过递归来表达::

    def quicksort(lst):
        "Quicksort over a list-like sequence"
        if len(lst) == 0:
            return lst
        pivot = lst[0]
        pivots = [x for x in lst if x == pivot]
        small = quicksort([x for x in lst if x < pivot])
        large = quicksort([x for x in lst if x > pivot])
        return small + pivots + large

函数体中使用的一些变量保存了中间结果，他们是不可变的(每次递归会创建新的栈空间，虽然变量名相同，
但并未改变上一个栈空间变量的指向)。如果我们想做，可以将定义写成单个的表达式，虽然影响可读性。
事实上，将它变成一个使用状态变量的迭代版本是有些困难的，并且比较晦涩。

作为一般建议，当一个问题看起来可以分割成为较小的问题时，尝试寻找递归表达的可能性是一个很好的做法，
特别是一个能避免状态变量和可变数据的版本。但大多数时候，在 Python 中，仅仅使用递归去完成迭代的功能
并不是一个好主意。
