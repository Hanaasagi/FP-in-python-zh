类方法
==========

所有的类方法都是可调用的。然而，在大多数情况下，调用实例方法违背了函数式编程的风格。通常我们使用方法，
是因为我们要引用绑定在实例属性中的可变数据，因此对方法的每次调用可能产生参数相同结果不同的情况。

**访问器和操作符**

访问器(使用 ``@property`` 或其他方法所创建)也是可调用的。尽管访问器在使用上是有限制的(从函数式
编程的角度)，因为它们的 getter 没有参数，setter 没有返回值::

    class Car(object):

        def __init__(self):
            self._speed = 100

        @property
        def speed(self):
            print("Speed is", self._speed)
            return self._speed

        @speed.setter
        def speed(self, value):
            print("Setting to", value)
            self._speed = value

    # >> car = Car()
    # >>> car.speed = 80   # Odd syntax to pass one argument
    # Setting to 80
    # >>> x = car.speed
    # Speed is 80

我们通过 Python 赋值语法来向访问器传入参数。这种操作对于 Python 来说相当容易::

    >>> class TalkativeInt(int):
            def __lshift__(self, other):
                print("Shift", self, "by", other)
                return int.__lshift__(self, other)
    ....
    >>> t = TalkativeInt(8)
    >>> t << 3
    Shift 8 by 3
    64

Python 中的每一个操作符对应着一个底层方法。虽然偶尔可能会产生可读性更高的 DSL，但为运算符定义特殊
的方法并没有改变调用底层函数的本质。

**实例的静态方法**

与函数式风格更为相近的一种类和方法的使用方法是是简单地将它们用作命名空间来存放各种相关的功能::

    import math
    class RightTriangle(object):
        "Class used solely as namespace for related functions"

        @staticmethod
        def hypotenuse(a, b):
            return math.sqrt(a**2 + b**2)

        @staticmethod
        def sin(a, b):
            return a / RightTriangle.hypotenuse(a, b)

        @staticmethod
        def cos(a, b):
            return b / RightTriangle.hypotenuse(a, b)

这样做避免了污染全局(或模块)命名空间，我们可以使用类名或相应的实例名来调用纯函数::

    >>> RightTriangle.hypotenuse(3,4)
    5.0
    >>> rt = RightTriangle()
    >>> rt.sin(3,4)
    0.6
    >>> rt.cos(3,4)
    0.8

目前为止，定义静态方法的最直接明显的方式是使用装饰器。但是，在 Python 3.x 中你也可以不使用装饰器，
"unbound method" 的概念已经退出舞台了。

    >>> import functools, operator
    >>> class Math(object):
    ...     def product(*nums):
    ...         return functools.reduce(operator.mul, nums)
    ...     def power_chain(*nums):
    ...         return functools.reduce(operator.pow, nums)
    ...
    >>> Math.product(3,4,5)
    60
    >>> Math.power_chain(3,4,5)
    3486784401

不过，若不使用 ``@staticmethod``，则无法从实例调用静态方法，因为他们仍然会隐式传入 ``self``::

    >>> m = Math()
    >>> m.product(3,4,5)
    -----------------------------------------------------------------
    TypeError
    Traceback (most recent call last)
    <ipython-input-5-e1de62cf88af> in <module>()
    ----> 1 m.product(3,4,5)

    <ipython-input-2-535194f57a64> in product(*nums)
          2 class Math(object):
          3     def product(*nums):
    ----> 4         return functools.reduce(operator.mul, nums)
          5     def power_chain(*nums):
          6         return functools.reduce(operator.pow, nums)

    TypeError: unsupported operand type(s) for *: 'Math' and 'int'

如果你的命名空间只是用来存放纯函数，那么没有理由不通过类来调用。但如果你想将纯函数和一些依赖于实例
可变状态的函数进行混合，那么你应当使用 ``@staticmethod`` 这个装饰器。

**生成器函数**

含有 ``yield`` 语句用来作为生成器的函数是 Python 中一类特殊的函数。调用它返回的不是一个普通的
值，而是一个能够被 ``next()`` 函数多次调用产生一序列值的迭代器。这将在惰性求值章节进行具体讨论。

虽然像 Python 的其他对象一样，存在着许多方法可以将状态引入到生成器中，但原则上生成器可以是纯的，
就像纯函数那样。它产生一序列值(可能无限)，而不是单一的一个值，但是仍然基于传入的参数。不过请注意，
生成器函数也有着内部状态；它是在调用签名(call signature)和返回值的边界，它们的行为就像一个无
副作用的“黑盒子”。一个简单的例子::

    >>> def get_primes():
    ...     "Simple lazy Sieve of Eratosthenes"
    ...     candidate = 2
    ...     found = []
    ...     while True:
    ...         if all(candidate % prime != 0 for prime in found):
    ...             yield candidate
    ...             found.append(candidate)
    ...         candidate += 1
    ...
    >>> primes = get_primes()
    >>> next(primes), next(primes), next(primes)
    (2, 3, 5)
    >>> for _, prime in zip(range(10), primes):
    ...     print(prime, end=" ")
    ....
    7 11 13 17 19 23 29 31 37 41

每次你通过 ``get_primes()`` 创建新对象，迭代器都是相同的无限惰性序列——另一个例子可能会传入
一些影响结果的值用于初始化——但对象本身是有状态的，正如它不断的被消费(consume)。
