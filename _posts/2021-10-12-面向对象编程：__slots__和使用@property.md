# 使用\_\_slots\_\_和@property

当我们创建好class（类），我们可以给这个类附加属性和方法。属性和方法分别对应函数式编程里面的参数和低阶函数。

尝试给实例绑定一个属性：

```python
class Student(object): # 创建类
    pass
    
>>> s = Student()      # 实例化类
>>> s.name = 'Michael' # 动态给实例绑定一个属性
>>> print(s.name)
Michael
```

还可以尝试给实例绑定一个方法：

```python
>>> def set_age(self, age): # 定义一个函数作为实例方法
...     self.age = age
...
>>> from types import MethodType
>>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
>>> s.set_age(25) # 调用实例方法
>>> s.age # 测试结果
25
```

注意的是，**上述这样先定义好类再绑定方法对另外一个实例是没有用的，如果想要对所有实例都有用，就要在定义类的时候定义好方法**

## \_\_slots\_\_

有时候我们定义好的类，不想外部输入太多属性，限制外部附加属性，就可以使用`__slots__`

比如，只允许对Student实例添加`name`和`age`属性。

为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的`__slots__`变量，来限制该class实例能添加的属性：

```
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

然后，我们试试：

```
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```

由于`'score'`没有被放到`__slots__`中，所以不能绑定`score`属性，试图绑定`score`将得到`AttributeError`的错误。

**使用`__slots__`要注意，`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的**

## @property

@property简单来说就是把方法转换为属性调用。这样做的目的是使得方法可以既能检查参数，又可以用类似属性这样简单的方式来访问类的变量。也可以防止属性被修改。

一般来讲，会先定义好`@property`将方法转换为属性，然后利用`@方法名字.setter`给属性赋值。

```python
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

**要特别注意：属性的方法名不要和实例变量重名。**例如，以下的代码是错误的：

```
class Student(object):

    # 方法名称和实例变量均为birth:
    @property
    def birth(self):
        return self.birth
```

这是因为调用`s.birth`时，首先转换为方法调用，在执行`return self.birth`时，又视为访问`self`的属性，于是又转换为方法调用，造成无限递归，最终导致栈溢出报错`RecursionError`。