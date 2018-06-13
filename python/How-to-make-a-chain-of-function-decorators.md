# [How to make a chain of function decorators?](https://stackoverflow.com/questions/739654/how-to-make-a-chain-of-function-decorators/)
## 描述
在python中如何使用两个装饰器达到下面输出的效果？
```
@makebold
@makeitalic
def say():
   return "Hello"
```
输出如下：
```
<b><i>Hello</i></b>
```
## 回答
如果你对详细的解释没有兴趣，可以直接看Paolo Bergantino的回答，如下
```
def makebold(fn):
    def wrapped():
        return "<b>" + fn() + "</b>"
    return wrapped

def makeitalic(fn):
    def wrapped():
        return "<i>" + fn() + "</i>"
    return wrapped

@makebold
@makeitalic
def hello():
    return "hello world"

print hello() ## returns "<b><i>hello world</i></b>"
```
### 装饰器基础
#### python的函数是对象
要理解装饰器，你必须先理解python里的函数是对象。这有着重要的影响。让我们先来看一个简单的例子
```
def shout(word="yes"):
    return word.capitalize()+"!"

print(shout())
# 输出 : 'Yes!'

# 作为对象，你可以把函数绑定给变量
scream = shout

print(scream())
# 输出 : 'Yes!'

# 更甚，你可以把旧的函数名'shout'删除，函数任然可以通过'scream'访问
del shout
try:
    print(shout())
except NameError as e:
    print(e)
    # 输出: "name 'shout' is not defined"

print(scream())
# 输出: 'Yes!
```
牢记这点，我们很快又会回到它。

另一个python函数有趣特性是它们可以在另一个函数中定义。
```
def talk():

    # 你可以在函数talk里随时定义一个函数
    def whisper(word="yes"):
        return word.lower()+"..."

    # 立刻调用函数!
    print(whisper())

# 每个你调用函数talk，内部会定义一个函数whisper，并在内部被调用
talk()
# 输出: "yes..."

# 但是函数whisper在函数talk外部是不存在的，即无法访问到
try:
    print(whisper())
except NameError as e:
    print(e)
    #输出 : "name 'whisper' is not defined"
```
#### 函数引用
看到了这里，你知道了函数：
+ 可以绑定给变量
+ 可以在另一个函数里定义

这意味着**一个函数可以返回另一个函数**
```
def getTalk(kind="shout"):

    # 定义两个函数
    def shout(word="yes"):
        return word.capitalize()+"!"

    def whisper(word="yes") :
        return word.lower()+"..."

    # 然后有选择的返回其中一个函数
    if kind == "shout":
        # 函数后面都没有带上括号，我们不是在调用函数，而是返回一个函数对象
        return shout  
    else:
        return whisper

# 我们怎么用这个奇怪的东西?

# 调用这个函数，并把结果绑定给一个变量
talk = getTalk()      

# 你可以看到talk是一个函数对象
print(talk)
# outputs : <function shout at 0xb7ea817c>

print(talk())
# outputs : Yes!

# 你也可以像下面这样直接使用:
print(getTalk("whisper")())
# outputs : yes...
```
如果你可以返回一个函数，那也可以将函数作为参数传递。
```
def doSomethingBefore(func): 
    print("I do something before then I call the function you gave me")
    print(func())

doSomethingBefore(scream)
# 输出: 
# I do something before then I call the function you gave me
# Yes!
```
好了，你已经掌握了理解装饰器需要的知识。装饰器就是“包装者”，意思是它可以让你在被装饰的函数被执行前或后运行额外的代码而不需要改变这个函数本身。
#### 手写的装饰器
```
# 一个装饰器就是一个函数，它将另一个函数作为参数执行
def my_shiny_new_decorator(a_function_to_decorate):

    # 在内部，装饰器定义了一个包装函数'the wrapper'，这个函数将包装在原始函数周围，
    # 于是便可以在运行原始函数前后后执行额外代码
    def the_wrapper_around_the_original_function():

        # 原始函数执行前要运行的代码
        print("Before the function runs")

        # 调用原始函数，使用括号
        a_function_to_decorate()

        # 原始函数执行后要运行的代码
        print("After the function runs")

    # 此刻，函数'a_function_to_decorate'还没有被执行
    # 我们返回刚创建的包装函数
    return the_wrapper_around_the_original_function

# 现在假设你创建了一个函数，之后你不会再修改它。
def a_stand_alone_function():
    print("I am a stand alone function, don't you dare modify me")

a_stand_alone_function() 
# 输出: I am a stand alone function, don't you dare modify me

# 你现在可以装饰它来扩展它的行为了，
# 只需把它传递给装饰器函数，返回一个装饰后的新函数
a_stand_alone_function_decorated = my_shiny_new_decorator(a_stand_alone_function)
a_stand_alone_function_decorated()
# 输出:
# Before the function runs
# I am a stand alone function, don't you dare modify me
# After the function runs
```
现在，你可以希望每次调用函数a_stand_alone_function，函数a_stand_alone_function_decorated会自动执行，很简单，用变量名'a_stand_alone_function'来绑定返回的新函数
```
a_stand_alone_function = my_shiny_new_decorator(a_stand_alone_function)
a_stand_alone_function()
# 输出:
# Before the function runs
# I am a stand alone function, don't you dare modify me
# After the function runs
```
#### 装饰器揭秘
上面的实例，使用装饰器语法：
```
@my_shiny_new_decorator
def another_stand_alone_function():
    print("Leave me alone")

another_stand_alone_function()  
# 输出:  
# Before the function runs
# Leave me alone
# After the function runs
```
是的，很简单，@decorator只是下面的一种快捷方式
```
another_stand_alone_function = my_shiny_new_decorator(another_stand_alone_function)
```
装饰器只是装饰器设计模式的python变体。在Python中，有几种典型的设计模式可以简化开发(比如迭代器)。

当然，你可以累加装饰器
```
def bread(func):
    def wrapper():
        print("</''''''\>")
        func()
        print("<\______/>")
    return wrapper

def ingredients(func):
    def wrapper():
        print("#tomatoes#")
        func()
        print("~salad~")
    return wrapper

def sandwich(food="--ham--"):
    print(food)

sandwich()
# 输出: --ham--
sandwich = bread(ingredients(sandwich))
sandwich()
# 输出:
# </''''''\>
# #tomatoes#
# --ham--
# ~salad~
# <\______/>
```
使用python装饰器语法
```
@bread
@ingredients
def sandwich(food="--ham--"):
    print(food)

sandwich()
# 输出:
# </''''''\>
# #tomatoes#
# --ham--
# ~salad~
# <\______/>
```
被装饰的顺序也很重要
```
@ingredients
@bread
def strange_sandwich(food="--ham--"):
    print(food)

strange_sandwich()
# 输出:
# #tomatoes#
# </''''''\>
# --ham--
# <\______/>
# ~salad~
```
### 现在来回答问题
作为结论，你可以很容易地看到如何回答这个问题:
```
def makebold(fn):
    def wrapper():
        return "<b>" + fn() + "</b>"
    return wrapper

def makeitalic(fn):
    def wrapper():
        return "<i>" + fn() + "</i>"
    return wrapper

@makebold
@makeitalic
def say():
    return "hello"

print(say())
# 输出: <b><i>hello</i></b>
```
你现在可以快乐的离开了，或者燃烧你的大脑，看到装饰者的高级用途。
### 装饰器的高级用法
#### 传递参数给被装饰的函数
```
# 这不是什么黑魔法，你只需让包装函数来传递参数

def a_decorator_passing_arguments(function_to_decorate):
    def a_wrapper_accepting_arguments(arg1, arg2):
        print("I got args! Look: {0}, {1}".format(arg1, arg2))
        function_to_decorate(arg1, arg2)
    return a_wrapper_accepting_arguments


# 因为当你调用装饰器返回的函数时，你实际在调用包装函数，所以传递参数给包装函数，就可以把参数转递给被装饰函数。
@a_decorator_passing_arguments
def print_full_name(first_name, last_name):
    print("My name is {0} {1}".format(first_name, last_name))

print_full_name("Peter", "Venkman")
# 输出:
# I got args! Look: Peter Venkman
# My name is Peter Venkman
```
#### 装饰类方法
Python的一个妙处是类方法和函数实际上是相同的。唯一的区别是，类方法期望它们的第一个参数是对当前对象(self)的引用。

这意味着您可以以相同的方式为方法构建装饰器:
```
def method_friendly_decorator(method_to_decorate):
    def wrapper(self, lie):
        lie = lie - 3 
        return method_to_decorate(self, lie)
    return wrapper


class Lucy(object):

    def __init__(self):
        self.age = 32

    @method_friendly_decorator
    def sayYourAge(self, lie):
        print("I am {0}, what did you think?".format(self.age + lie))

l = Lucy()
l.sayYourAge(-3)
# 输出: I am 26, what did you think?
```
如果你是想创建通用装饰器-即无需考虑参数，可以应用到任何函数或类方法上，那么只需使用*args, **kwrags
```
def a_decorator_passing_arbitrary_arguments(function_to_decorate):
    # 这个包装函数接受任何参数
    def a_wrapper_accepting_arbitrary_arguments(*args, **kwargs):
        print("Do I have args?:")
        print(args)
        print(kwargs)
        # 然后你解包参数， *args, **kwargs
        function_to_decorate(*args, **kwargs)
    return a_wrapper_accepting_arbitrary_arguments

@a_decorator_passing_arbitrary_arguments
def function_with_no_argument():
    print("Python is cool, no argument here.")

function_with_no_argument()
# 输出
# Do I have args?:
# ()
# {}
# Python is cool, no argument here.

@a_decorator_passing_arbitrary_arguments
def function_with_arguments(a, b, c):
    print(a, b, c)

function_with_arguments(1,2,3)
# 输出
# Do I have args?:
# (1, 2, 3)
# {}
# 1 2 3 

@a_decorator_passing_arbitrary_arguments
def function_with_named_arguments(a, b, c, platypus="Why not ?"):
    print("Do {0}, {1} and {2} like platypus? {3}".format(a, b, c, platypus))

function_with_named_arguments("Bill", "Linus", "Steve", platypus="Indeed!")
# 输出
# Do I have args ? :
# ('Bill', 'Linus', 'Steve')
# {'platypus': 'Indeed!'}
# Do Bill, Linus and Steve like platypus? Indeed!

class Mary(object):

    def __init__(self):
        self.age = 31

    @a_decorator_passing_arbitrary_arguments
    def sayYourAge(self, lie=-3):
        print("I am {0}, what did you think?".format(self.age + lie))

m = Mary()
m.sayYourAge()
# 输出
# Do I have args?:
# (<__main__.Mary object at 0xb7d303ac>,)
# {}
# I am 28, what did you think?
```
#### 传递参数给装饰器
好了，现在你可能要问如何传递参数给装饰器自身？

这变得有些复杂，因为一个装饰器必须接受一个函数作为参数，因此你不能直接将被装饰函数的参数传递给装饰器。

在快速解决方案之前，让我们写一个小提示:
```
# 装饰器是普通的函数
def my_decorator(func):
    print("I am an ordinary function")
    def wrapper():
        print("I am function returned by the decorator")
        func()
    return wrapper

# 因此你可以不使用@来调用它

def lazy_function():
    print("zzzzzzzz")

decorated_function = my_decorator(lazy_function)
# 输出: I am an ordinary function

@my_decorator
def lazy_function():
    print("zzzzzzzz")

# 输出: I am an ordinary function
```
完全一样。'my_decorator'被调用，当你使用@my_decorator时，你告诉python取执行由变量'my_decorator'标记的函数。

这是重要的！你给的标签可以直接指向装饰器，或不是。

让我们更“邪恶”一些
```
def decorator_maker():

    print("I make decorators! I am executed only once: "
          "when you make me create a decorator.")

    def my_decorator(func):

        print("I am a decorator! I am executed only when you decorate a function.")

        def wrapped():
            print("I am the wrapper around the decorated function. "
                  "I am called when you call the decorated function. "
                  "As the wrapper, I return the RESULT of the decorated function.")
            return func()

        print("As the decorator, I return the wrapped function.")

        return wrapped

    print("As a decorator maker, I return a decorator")
    return my_decorator

# 让我们创建一个装饰器
new_decorator = decorator_maker()       
# 输出:
# I make decorators! I am executed only once: when you make me create a decorator.
# As a decorator maker, I return a decorator

# 然后我们来装饰函数

def decorated_function():
    print("I am the decorated function.")

decorated_function = new_decorator(decorated_function)
# 输出:
# I am a decorator! I am executed only when you decorate a function.
# As the decorator, I return the wrapped function

# 调用函数:
decorated_function()
# 输出:
# I am the wrapper around the decorated function. I am called when you call the decorated function.
# As the wrapper, I return the RESULT of the decorated function.
# I am the decorated function.
```
让我们做同样的事情，但是跳过所有讨厌的中间变量:
```
def decorated_function():
    print("I am the decorated function.")
decorated_function = decorator_maker()(decorated_function)
# 输出:
# I make decorators! I am executed only once: when you make me create a decorator.
# As a decorator maker, I return a decorator
# I am a decorator! I am executed only when you decorate a function.
# As the decorator, I return the wrapped function.

decorated_function()    
# 输出:
# I am the wrapper around the decorated function. I am called when you call the decorated function.
# As the wrapper, I return the RESULT of the decorated function.
# I am the decorated function.
```
让代码变得更精简一些
```
@decorator_maker()
def decorated_function():
    print("I am the decorated function.")
# 输出:
# I make decorators! I am executed only once: when you make me create a decorator.
# As a decorator maker, I return a decorator
# I am a decorator! I am executed only when you decorate a function.
# As the decorator, I return the wrapped function.

decorated_function()    
# 输出:
# I am the wrapper around the decorated function. I am called when you call the decorated function.
# As the wrapper, I return the RESULT of the decorated function.
# I am the decorated function.
```
回到装饰器的参数问题，如果我们可以使用函数生成装饰器，那么我们可以把参数传递给那个函数。
```
def decorator_maker_with_arguments(decorator_arg1, decorator_arg2):

    print("I make decorators! And I accept arguments: {0}, {1}".format(decorator_arg1, decorator_arg2))

    def my_decorator(func):
        # 这里传递参数的能力来自于闭包
        print("I am the decorator. Somehow you passed me arguments: {0}, {1}".format(decorator_arg1, decorator_arg2))

        # 不要混淆装饰器的参数和被装饰函数的参数
        def wrapped(function_arg1, function_arg2) :
            print("I am the wrapper around the decorated function.\n"
                  "I can access all the variables\n"
                  "\t- from the decorator: {0} {1}\n"
                  "\t- from the function call: {2} {3}\n"
                  "Then I can pass them to the decorated function"
                  .format(decorator_arg1, decorator_arg2,
                          function_arg1, function_arg2))
            return func(function_arg1, function_arg2)

        return wrapped

    return my_decorator

@decorator_maker_with_arguments("Leonard", "Sheldon")
def decorated_function_with_arguments(function_arg1, function_arg2):
    print("I am the decorated function and only knows about my arguments: {0}"
           " {1}".format(function_arg1, function_arg2))

decorated_function_with_arguments("Rajesh", "Howard")
print('*'*20)
decorated_function_with_arguments("1", "2")
# 输出:
# I make decorators! And I accept arguments: Leonard Sheldon
# I am the decorator. Somehow you passed me arguments: Leonard Sheldon
# I am the wrapper around the decorated function. 
# I can access all the variables 
#        - from the decorator: Leonard Sheldon 
#        - from the function call: Rajesh Howard 
# Then I can pass them to the decorated function
# I am the decorated function and only knows about my arguments: Rajesh Howard
# ********************
# I am the wrapper around the decorated function.
# I can access all the variables
#        - from the decorator: Leonard Sheldon
#        - from the function call: 1 2
# Then I can pass them to the decorated function
# I am the decorated function and only knows about my arguments: 1 2
```
如你所见，你可以使用这个技巧传递参数给装饰器。你甚至可以使用*args, **kwargs。但是记住装饰器只被调用一次。
当您执行"import x"时，函数已经被修饰过了，之后不能动态设置参数，所以您不能更改任何内容。
### 练习一下：装饰一个装饰器
让我们来点乐趣，装饰一个装饰器。
```
def decorator_with_args(decorator_to_enhance):
    """ 
    这个函数被当做装饰器来使用
    它允许任何装饰器接受一定数量的参数
    """

    # 我们使用相同的技巧传递参数
    def decorator_maker(*args, **kwargs):

        # 我们动态地创建一个装饰,只接受一个函数,但从maker传递参数。
        def decorator_wrapper(func):

            # 我们返回原始装饰器的结果，毕竟只是一个普通的函数(返回一个函数)，
            # 唯一的缺陷:装饰者必须有这个特定的签名，否则它将不起作用:
            return decorator_to_enhance(func, *args, **kwargs)

        return decorator_wrapper

    return decorator_maker
```
你可以像下面使用它：
```
# 创建一个将要使用的装饰器函数，并在上面“贴”一个装饰器
# 记住装饰器函数的签名是"decorator(func, *args, **kwargs)"
@decorator_with_args 
def decorated_decorator(func, *args, **kwargs): 
    def wrapper(function_arg1, function_arg2):
        print("Decorated with {0} {1}".format(args, kwargs))
        return func(function_arg1, function_arg2)
    return wrapper

# 然后你就可以用你的新装修装饰师来装饰你想要的功能了。

@decorated_decorator(42, 404, 1024)
def decorated_function(function_arg1, function_arg2):
    print("Hello {0} {1}".format(function_arg1, function_arg2))

decorated_function("Universe and", "everything")
# 输出:
# Decorated with (42, 404, 1024) {}
# Hello Universe and everything
```
### 最佳实践
fuctools模块包含函数functools.wraps()，它可以将被装饰函数的name,module和docstring拷贝到它的包装者。
（实际上functools.wraps()就是一个装饰器）
```
def foo():
    print("foo")

print(foo.__name__)
# 输出: foo

# 使用装饰器后，情况变混乱了   
def bar(func):
    def wrapper():
        print("bar")
        return func()
    return wrapper

@bar
def foo():
    print("foo")

print(foo.__name__)
# 输出: wrapper

# "functools"可以改变这种情况

import functools

def bar(func):
    @functools.wraps(func)
    def wrapper():
        print("bar")
        return func()
    return wrapper

@bar
def foo():
    print("foo")

print(foo.__name__)
# 输出: foo
```



