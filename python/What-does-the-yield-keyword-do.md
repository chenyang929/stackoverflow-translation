# [What does the "yield" keyword do?](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do/)
## 描述
见原始页面
## 回答
为了理解"yield"的作用，你必须理解生成器，而之前你要先理解迭代器。
### 迭代器
当你创建一个列表后，可以逐次读取列表中的每一项。这个读取过程就叫做迭代。
```
>>> mylist = [1, 2, 3]
>>> for i in mylist:
...    print(i)
1
2
3
```
mylist是一个迭代器。你可以使用列表推导式创建列表，也生成迭代器。
```
>>> mylist = [x*x for x in range(3)]
>>> for i in mylist:
...    print(i)
0
1
4
```
任何你可以使用"for ... in..."的对象都是一个迭代器，例如lists,strings,files...

这些迭代器很方便，因为您可以随心所欲地读取它们，但是您将所有的值存储在内存中，当您有很多值时，这并不总是您想要的。
### 生成器
生成器是迭代器--一种只能迭代一次的迭代器。生成器不把所有值全部存在内存里，而是随用随生成。
```
>>> mygenerator = (x*x for x in range(3))
>>> for i in mygenerator:
...    print(i)
0
1
4
```
只是用圆括号代替列表推导式的方括号，但是你不能第二次使用"for i in mygenerator"，因为生成器只能被使用一次：计算生成0，然后丢弃，然后是1，最后是4。
### yield
yield是一个类似return的关键字，除了函数将返回一个生成器。
```
>>> def createGenerator():
...    mylist = range(3)
...    for i in mylist:
...        yield i*i
...
>>> mygenerator = createGenerator() # create a generator
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```
这是一个没用的例子，但是当你知道你的函数会返回一大堆你只需要读一次的值时，这个例子很有用。

为了掌握yield，你必须理解当你调用函数时，函数内的代码不会运行，而是返回一个可迭代对象。

然后代码会在每次for循环时运行，下面是最难理解的部分：

函数返回的生成器对象第一次for循环，函数中的代码会从头开始运行，直到遇到yield，然后它将返回循环的第一个值。然后，下一次for循环将从第一次停止处开始运行，直到再次遇到yield，并返回下一个值，该过程持续直到没有返回值为止。

当函数运行到没有yield时，生成器被认为是空的。这可能是因为循环已经结束，或者因为您不再满足“if/else”。