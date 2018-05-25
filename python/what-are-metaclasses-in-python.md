# [what are mataclasses in python](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python/)
> python中的元类是什么？可以做什么？
## 作为对象的类(Classes as objects)
在理解元类之前，你需要先掌握python中的类。Python有一个非常独特的概念关于什么是类，这是从Smalltalk语言借鉴来的。

在大多数语言中，类只是描述如何生成对象的代码片段。这在Python中也是一样的:
<pre><code>
>>> class ObjectCreator(object): # pyhton3无需显示指定object
...       pass
...

>>> my_object = ObjectCreator()
>>> print(my_object)
<__main__.ObjectCreator object at 0x8974f2c>
</pre></code>
但是在python中类的意义远不止如此，类也是对象。是的，是对象！

一旦你用关键字class，python就会执行它并创建一个对象。上面的代码就会在内存中创建一个名字叫“ObjectCreator”的对象。而这个对象(类)本身能够创建对象(实例)，这就是为什么它是类的原因。

由于它是对象，便可以：
+ 绑定给一个变量
+ 复制
+ 添加属性
+ 作为函数参数

例如：
<pre><code>
>>> print(ObjectCreator) # you can print a class because it's an object
<class '__main__.ObjectCreator'>
>>> def echo(o):
...       print(o)
...
>>> echo(ObjectCreator) # you can pass a class as a parameter
<class '__main__.ObjectCreator'>
>>> print(hasattr(ObjectCreator, 'new_attribute'))
False
>>> ObjectCreator.new_attribute = 'foo' # you can add attributes to a class
>>> print(hasattr(ObjectCreator, 'new_attribute'))
True
>>> print(ObjectCreator.new_attribute)
foo
>>> ObjectCreatorMirror = ObjectCreator # you can assign a class to a variable
>>> print(ObjectCreatorMirror.new_attribute)
foo
>>> print(ObjectCreatorMirror())
<__main__.ObjectCreator object at 0x8997b4c>
</pre></code>
## 动态的创建类(Creating classess dynamically)
由于类是对象，你可以随时的创建它们，就像其他任何对象一样。

首先，你可以在一个函数中使用class创建一个类：
<pre><code>
>>> def choose_class(name):
...     if name == 'foo':
...         class Foo(object):
...             pass
...         return Foo # return the class, not an instance
...     else:
...         class Bar(object):
...             pass
...         return Bar
...
>>> MyClass = choose_class('foo')
>>> print(MyClass) # the function returns a class, not an instance< class '__main__.Foo'>
>>> print(MyClass()) # you can create an object from this class
<__main__.Foo object at 0x89c6d4c>
</pre></code>
但这还不够动态，因为你仍然需要写完整的类。

由于类是对象，那它一定是有什么东西生成的。

当你使用class关键字，python自动创建这个对象。但就像python中的大多数东西一样，你可以手动的执行这个操作。

记得type函数吗？它可以告诉你一个对象的类型：
<pre><code>
>>> print(type(1))
< type 'int'>
>>> print(type("1"))
< type 'str'>
>>> print(type(ObjectCreator))
< type 'type'>
>>> print(type(ObjectCreator()))
< class '__main__.ObjectCreator'>
</pre></code>
嗯，type还有一个完全不同的能力，它可以创建类。type可以接受一个类的描述作为参数，然后返回一个类。

（我知道，同样的函数可以根据你传递给它的参数不同而有两个完全不同的用途，这是愚蠢的。由于Python的向后兼容性，这是一个问题。）

type像如下方式创建类
<pre><code>
type(name of the class,
     tuple of the parent class (for inheritance, can be empty),
     dictionary containing attributes names and values)
</pre></code>
例如：
<pre><code>
>>> class MyShinyClass(object):
...       pass
</pre></code>
可以这样手动创建:
<pre><code>
>>> MyShinyClass = type('MyShinyClass', (), {}) # returns a class object
>>> print(MyShinyClass)
< class '__main__.MyShinyClass'>
>>> print(MyShinyClass()) # create an instance with the class
<__main__.MyShinyClass object at 0x8997cec>
</pre></code>
你注意到我使用“MyShinyClass”作为类和绑定这个类的变量的名字。它们可以不同，但没理由把事情弄复杂。

type接受一个字典来定义类的属性，故：
<pre><code>
>>> class Foo(object):
...       bar = True
</pre></code>
可以转换成：
<pre><code>
>>> Foo = type('Foo', (), {'bar':True})
</pre></code>
并且像一般的类一样使用：
<pre><code>
>>> print(Foo)
< class '__main__.Foo'>
>>> print(Foo.bar)
True
>>> f = Foo()
>>> print(f)
<__main__.Foo object at 0x8a9b84c>
>>> print(f.bar)
True
</pre></code>
当然它也可以被继承，通常类的继承如下
<pre><code>
>>>   class FooChild(Foo):
...         pass
</pre></code>
可以写成
<pre><code>
>>> FooChild = type('FooChild', (Foo,), {})
>>> print(FooChild)
< class '__main__.FooChild'>
>>> print(FooChild.bar) # bar is inherited from Foo
True
</pre></code>
你可能想给你的类添加方法。只需用适当的签名定义一个函数并将其赋值为一个属性。
<pre><code>
>>> def echo_bar(self):
...       print(self.bar)
...
>>> FooChild = type('FooChild', (Foo,), {'echo_bar': echo_bar})
>>> hasattr(Foo, 'echo_bar')
False
>>> hasattr(FooChild, 'echo_bar')
True
>>> my_foo = FooChild()
>>> my_foo.echo_bar()
True
</pre></code>
就像给给一般的类添加方法一样，你可以在动态的创建完类后再给它添加更多的方法。
<pre><code>
>>> def echo_bar_more(self):
...       print('yet another method')
...
>>> FooChild.echo_bar_more = echo_bar_more
>>> hasattr(FooChild, 'echo_bar_more')
True
</pre></code>
您现在知道的是:在Python中，类是对象，您可以动态地创建一个类。

当您使用关键字class时，这就是Python所做的事情，它通过使用元类来实现这一点。
## 什么是元类(What are metaclasses)
元类就是创建类的“员工”。

你定义类就是为了创建对象，不是吗？

但是我们知道python中的类就是对象。

好吧，元类就是创建这些对象的。它们是“类的类”，你可以这样想象它们。
<pre><code>
MyClass = MetaClass()
my_object = MyClass()
</pre></code>
你已经知道type允许你做下面的事情
<pre><code>
MyClass = type('MyClass', (), {})
</pre></code>
这是因为type事实上就是一个元类，type是python在幕后使用创建所有类的元类。

现在你可能想知道它为什么是小写，而不是Tpye？

嗯，我猜是为了统一，str是创建字符串类型对象的类，int是创建整数型对象的类，而type是创建类对象的类。

通过检查__class__属性可以看到这一点。

所有东西在python中都是一个对象，包括ints,strings,functions和classes。它们都是对象，都创建自一个类。
<pre><code>
>>> age = 35
>>> age.__class__
< type 'int'>
>>> name = 'bob'
>>> name.__class__
< type 'str'>
>>> def foo(): pass
>>> foo.__class__
< type 'function'>
>>> class Bar(object): pass
>>> b = Bar()
>>> b.__class__
< class '__main__.Bar'>
</pre></code>
那么"__ class __" 的 " __ class __"是什么？
<pre><code>
>>> age.__class__.__class__
< type 'type'>
>>> name.__class__.__class__
< type 'type'>
>>> foo.__class__.__class__
< type 'type'>
>>> b.__class__.__class__
< type 'type'>
</pre></code>
所以一个元类就是创建类对象的“员工”。如果愿意你也可以叫它“类工厂”。

type是python使用的内建元类，当然你可以创建你自己的元类。
## 定制元类(Custom metaclasses)
元类的主要用途是在类创建时自动的更改类。

你通常在使用APIs，即你想创建匹配当前上下文的类时，会用到元类。

想象一个愚蠢的例子，你想让你模块中的类的属性名全部大写。有几种方式可是实现，其中一个便是使用元类。

方法是，所有的类通过这个元类来创建，元类将属性名变成大写。
<pre><code>
class UpperAttrMetaclass(type):

    def __new__(cls, clsname, bases, dct):

        uppercase_attr = {}
        for name, val in dct.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return type.__new__(cls, clsname, bases, uppercase_attr)
        #return super(UpperAttrMetaclass, cls).__new__(cls, clsname, bases, uppercase_attr) # py2和py3
        #return super().__new__(cls, clsname, bases, uppercase_attr) # py3

class Foo(metaclass=UpperAttrMetaclass):
    bar = 'bip'

print(hasattr(Foo, 'bar'))
# Out: False
print(hasattr(Foo, 'BAR'))
# Out: True

f = Foo()
print(f.BAR)
# Out: 'bip'
</pre></code>
这些就是全部关于python元类的东西了。

使用元类的代码复杂性背后的原因并不是因为元类，而是因为您通常使用元类来执行依赖于内省、操纵继承、__dict__等变量之类的复杂事情。

的确，元类对黑魔法特别有用，因此也很复杂。但他们自己却很简单:
+ 劫持类的创建
+ 修改类
+ 返回修改后的类
## 最后的话
你已经知道类是对象并且可以创建其实例。

实际上，类本身是元类的实例。
<pre><code>
>>> class Foo(object): pass
>>> id(Foo)
142630324
</pre></code>
python中所有东西都是对象，它们要么是类的实例，要么是元类的实例。

除了type。

type是它自身的元类。这不是您可以在纯Python中重新生成的东西，而是通过在实现级别上的一些欺骗来完成的。
其次,元类是复杂的。您可能不希望使用它们进行非常简单的类修改。您可以使用两种不同的技术来改变类:
+ 猴子补丁
+ 类装饰器
### 小例子
我们知道python的列表有append方法，我们来实现一个自己的列表，它有add方法，其实就是append的别名而已。
<pre><code>
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)

class CustomList(list, metaclass=ListMetaclass):
    pass

lst = CustomList()
lst.add('custom_list_1')
lst.add('custom_list_2')

lst
['custom_list_1', 'custom_list_2']
</pre></code>










