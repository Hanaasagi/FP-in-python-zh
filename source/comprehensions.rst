推导
=======

使用推导通常是一个既能使代码更紧凑，也能将我们的焦点从 "how" 转移到 "what" 的方式。
推导是一个使用和循环与分支操作相同关键字的表达式，但将注意力从过程转移到了数据本身。
简单地改写表达的形式一般会使我们对代码的理解以及理解的容易程度有很大的不同。三位运算符
(if-else 表达式)使用了不同顺序的关键字，也表现出了类似的对于关注点的重组。假设我们的代码是::

    collection = list()
    for datum in data_set:
        if condition(datum):
            collection.append(datum)
        else:
            new = modify(datum)
            collection.append(new)

我们可以写成更紧凑的形式::

    collection = [d if condition(d) else modify(d)
                 for d in data_set]

比起简单地保存几个字符和行更重要的是通过思考何为 ``collection`` 所带来的心理转变，
避免去考虑或调试 ``collection`` 在循环中某一处的状态。

列表推导是 Python 中最长的(longest???)，在某些方面是最简易的。 Python 现在也支持生成器推导，
集合推导和字典推导语法。作为一个警告，虽然你可以将推导进行任意深度的嵌套，但越过某个界限后，
往往会开始模糊。对于真正复杂的数据集合构建，重构成为函数，可读性会更高。


**生成器**

生成器推导和列表推导虽然有着相似的语法(生成器不使用方括号包裹，在某些上下文中需要使用圆括号包裹)，
但生成器是惰性的。也就是说，若没有调用对象的 `.next()` 方法，它们仅仅是对于如何获取数据的一种描述。
直到真正地被需要前，节省了长序列和延迟计算的内存。例如::

    log_lines = (line for line in read_line(huge_log_file)
                      if complex_condition(line))

生成器推导也能够构造列表，但运行时更加友好。生成器推导也有着命令式的版本::

    def get_log_lines(log_file):
        line = read_line(log_file)
        while True:
            try:
                if complex_condition(line):
                    yield line
                line = read_line(log_file)
            except StopIteration:
                raise

    log_lines = get_log_lines(huge_log_file)


命令式的版本也可以简化，但展示这个版本是为了举例说明 ``for`` 循环迭代一个可迭代对象的
幕后机制，那些我们从想法中抽象出的诸多细节。事实上，使用 ``yield`` 也是一种对于底层
迭代器协议(iterator protocol)的抽象。我们也能使用实现 ``.__next__()`` 和 ``.__iter__()``
方法的类来达成目的::

    class GetLogLines(object):

        def __init__(self, log_file):
            self.log_file = log_file
            self.line = None

        def __iter__(self):
            return self

        def __next__(self):
            if self.line is None:
                self.line = read_line(log_file)
            while not complex_condition(self.line):
                self.line = read_line(self.log_file)
            return self.line

    log_lines = GetLogLines(huge_log_file)

抛开离题的迭代器协议和惰性部分，读者应当看到推导将注意力更多地集中到 "What"，而命令式版本尽管
重构的非常成功，也依然在关注 "How"。

**字典和集合**

列表可以直接在推导中创建而不是先创建一个空列表，然后通过循环重复调用 ``.append()``；类似地，
字典和集合也能够一次性创建，而不是在循环中重复调用 ``.update()`` 或 ``.add()``，例如::

    >>> {i:chr(65+i) for i in range(6)}
    {0: 'A', 1: 'B', 2: 'C', 3: 'D', 4: 'E', 5: 'F'}
    >>> {chr(65+i) for i in range(6)}
    {'A', 'B', 'C', 'D', 'E', 'F'}

这些推导的命令式版本实现和之前所展示的例子相似。
