多分派(Multiple Dispatch)
===========================

编写多路执行代码的一个非常有趣的方法是通过名为 "multiple dispatch"("multimethods") 的技术。
这种技术为单个函数声明多个签名，并调用与参数的类型或属性相匹配的函数。这种技术通常允许我们去减少或
避免显示分支条件的使用，使用更直观的参数模式描述来作为替代。

很久之前，作者写了一个名叫 ``multimethods`` 的模块，它能够通过选项来灵活的进行线性分发，但它
只能运行在 Python 2.x 上，并且是在 Python 提出装饰器这个优雅概念之前写的。Matthew Rocklin’s
more recent ``multipledispatch`` is a modern approach for recent Python versions,
albeit it lacks some of the theoretical arcana I explored in my ancient module.
理想情况下，在作者看来，未来的 Python 应当包含能进行多分派的标准语法或 API(但有可能这个任务会由
第三方库来完成)

为了解释多派发如何能使代码变得可读性更好并且较少发生错误，让我们用三种方式来实现剪刀/石头/布游戏。
下面的类是三个版本共有的::

    class Thing(object): pass
    class Rock(Thing): pass
    class Paper(Thing): pass
    class Scissors(Thing): pass

**多分支**

首先是纯命令版本，这会有许多重复的，嵌套的，条件块，会令人出错::

    def beats(x, y):
        if isinstance(x, Rock):
            if isinstance(y, Rock):
                return None           # No winner
            elif isinstance(y, Paper):
                return y
            elif isinstance(y, Scissors):
                return x
            else:
                raise TypeError("Unknown second thing")
        elif isinstance(x, Paper):
            if isinstance(y, Rock):
                return x
            elif isinstance(y, Paper):
                return None           # No winner
            elif isinstance(y, Scissors):
                return y
            else:
                raise TypeError("Unknown second thing")
        elif isinstance(x, Scissors):
            if isinstance(y, Rock):
                return y
            elif isinstance(y, Paper):
                return x
            elif isinstance(y, Scissors):
                return None          # No winner
            else:
                raise TypeError("Unknown second thing")
        else:
            raise TypeError("Unknown first thing")

    rock, paper, scissors = Rock(), Paper(), Scissors()
    # >>> beats(paper, rock)
    # <__main__.Paper at 0x103b96b00>
    # >>> beats(paper, 3)
    # TypeError: Unknown second thing

**委托给对象**

作为第二个尝试，我们会使用 Python 的 "duck typing" 来消除一些不必要的重复， "duck typing"
指我们比起类型更关注其所具有的方法，含有相同的方法就可以被调用，尽管它们的类型可能不同::

    class DuckRock(Rock):
        def beats(self, other):
            if isinstance(other, Rock):
                return None         # No winner
            elif isinstance(other, Paper):
                return other
            elif isinstance(other, Scissors):
                return self
            else:
                raise TypeError("Unknown second thing")

    class DuckPaper(Paper):
        def beats(self, other):
            if isinstance(other, Rock):
                return self
            elif isinstance(other, Paper):
                return None       # No winner
            elif isinstance(other, Scissors):
                return other
            else:
                raise TypeError("Unknown second thing")

    class DuckScissors(Scissors):
        def beats(self, other):
            if isinstance(other, Rock):
                return other
            elif isinstance(other, Paper):
                return self
            elif isinstance(other, Scissors):
                return None     # No winner
            else:
                raise TypeError("Unknown second thing")

    def beats2(x, y):
        if hasattr(x, 'beats'):
            return x.beats(y)
        else:
            raise TypeError("Unknown first thing")

    rock, paper, scissors = DuckRock(), DuckPaper(), DuckScissors()
    # >>> beats2(rock, paper)
    # <__main__.DuckPaper at 0x103b894a8>
    # >>> beats2(3, rock)
    # TypeError: Unknown first thing

我们实际上并没有减少代码量，但是这个版本降低了每个 callable 的复杂性，并减少了条件语句的嵌套层级。
大多数的逻辑被放入独立的类中，而不是更深层次的分支。在面向对象编程中，我们可以将分派委托到对象
(but only to the one controlling object)。

**模式匹配**

作为最后的尝试，我们可以使用多分派(multiple dispatch)来更直接地表达所有的逻辑。这会更加易读，
尽管仍然有一定量的分支::

    from multipledispatch import dispatch

    @dispatch(Rock, Rock)
    def beats3(x, y): return None

    @dispatch(Rock, Paper)
    def beats3(x, y): return y

    @dispatch(Rock, Scissors)
    def beats3(x, y): return x

    @dispatch(Paper, Rock)
    def beats3(x, y): return x

    @dispatch(Paper, Paper)
    def beats3(x, y): return None

    @dispatch(Paper, Scissors)
    def beats3(x, y): return x

    @dispatch(Scissors, Rock)
    def beats3(x, y): return y

    @dispatch(Scissors, Paper)
    def beats3(x, y): return x

    @dispatch(Scissors, Scissors)
    def beats3(x, y): return None

    @dispatch(object, object)
    def beats3(x, y):
        if not isinstance(x, (Rock, Paper, Scissors)):
            raise TypeError("Unknown first thing")
        else:
            raise TypeError("Unknown second thing")

    # >>> beats3(rock, paper)
    # <__main__.DuckPaper at 0x103b894a8>
    # >>> beats3(rock, 3)
    # TypeError: Unknown second thing

**基于谓词的分派(Predicate-Based Dispatch)**

使用分派来表达条件语句的一种奇葩方式是直接在函数签名中包含谓词(或者对函数使用装饰器，像
``multipledispatch``)。我目前不知道有这样的 Python 库，但让我们定一个假想的库来简单的说明这个
概念。这个假想库叫做 ``predicative_dispatch``::

    from predicative_dispatch import predicate

    @predicate(lambda x: x < 0, lambda y: True)
    def sign(x, y):
        print("x is negative; y is", y)

    @predicate(lambda x: x == 0, lambda y: True)
    def sign(x, y):
        print("x is zero; y is", y)

    @predicate(lambda x: x > 0, lambda y: True)
    def sign(x, y):
        print("x is positive; y is", y)

虽然这个小例子显然不是一个完整的规范，但读者能看到我们如何将大部分甚至全部条件语句移至函数调用签名中，
这会导致更小，更容易理解和调试的函数。

*译者注，使用策略模式也可以减少 if-else 的嵌套层级*
