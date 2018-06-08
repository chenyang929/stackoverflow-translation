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
