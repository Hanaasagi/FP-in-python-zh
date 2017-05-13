封装
=======
一个将注意力放在 "how" 而不是 "what" 的明显途径是重构代码，将数据构建
放在一个相对隔离的地方，比如函数或方法内部。举个例子，考虑想下面这样的
命令式代码片段::

    # configure the data to start with
    collection = get_initial_state()
    state_var = None
    for datum in data_set:
        if condition(state_var):
            state_var = calculate_from(datum)
            new = modify(datum, state_var)
            collection.add_to(new)
        else:
            new = modify_differently(datum)
            collection.add_to(new)

    # Now actually work with the data
    for thing in collection
        process(thing)

我们可以简单地从当前的作用域中移除数据的构建("how")，然后将他们挤进一个
可以独立考虑的函数中（进行充分的抽象）。 例如::

    # tuck away construction of data
    def make_collection(data_set):
        collection = get_initial_state()
        state_var = None
        for datum in data_set:
            if condition(state_var):
                state_var = calculate_from(datum, state_var)
                new = modify(datum, state_var)
                collection.add_to(new)
            else:
                new = modify_differently(datum)
                collection.add_to(new)
        return collection
    # Now actually work with the data
    for thing in make_collection(data_set):
        process(thing)

我们没有改变程序的逻辑，但我们已经将焦点从如何构造 ``collection`` 转移到
``make_collection()`` 创建了什么
