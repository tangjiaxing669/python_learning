# Three Day

***

### 编程范式

- 编程范式指的是软件工程中的一种方法学.
- 常见的编程范式有：OOP(面向对象编程)、FP(函数式编程)、PP(过程编程)、IP(指令编程)
- 通常每种语言都会提倡一些范式，也有的语言支持多种范式
- Python是一种多范式语言，但是对OOP支持最好

### OOP基本哲学

- 世界是由对象组成的
- 对象具有运动规律和内部状态
- 对象之间的相互作用和通讯构成世界

### 对象的特性

- 唯一性：世界上没有两片相同的树叶
- 分类型：分类是对现实世界的抽象

### OOP的三大特征

- 继承
- 多态
- 封装

### 面向对象的本质

- 面向对象是对数据和行为的封装
- 但是有时候，数据仅仅是数据，方法仅仅是方法

先来看个列子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class MyClass(object):
    # 注意，所有类实例都是共享类变量的初值的，并且类变量的初值是可以改变的，可以通过 instance_1.val += 1 等方法来改变
    flag = 1        # 类变量；我们可以通过 MyClass.flag 来访问类变量；也可以通过 instance_1 = MyClass(3,4); instance_1.flag 来操作这个变量；
    __pri_flag = 2  # 私有的类变量，类外部不能直接访问

    def __init__(self, x, y):   # 构造函数(初始化函数)，仅用来初始化实例变量；里面的 x 和 y 都是实例变量
        self.x = x      # 比如我们实例化一个 instance_1 = MyClass(3,4)，那么可以通过 instance_1.x 来直接访问，还可以通过 instance_1.x = 5 赋值
        self.__y = y    # __y 表明这是一个私有属性，不能通过实例名直接访问，也不能执行赋值操作，只能在类或实例中进行操作

    def instance_method(self):  # 实例方法
        print('Instance_method for self.x => {0}'.format(self.x))
        print('Instance_method for self.__y => {0}'.format(self.__y))

    def __pri_instance_method(self):    # 以双下划线 __ 开头的，被称为实例的私有方法；实例中以双下划线 __ 开头的变量，被称为实例的私有属性
        print('__pri_instance_method for self.x => {0}'.format(self.x))
        print('__pri_instance_method for self.__y => {0}'.format(self.__y))

    @classmethod
    def class_method(cls):     # 类方法；类方法必须要传递一个变量，这个变量就是类对象的本身；然后通过类本身去调用类变量
        print('class_method for self.x => {0}'.format(cls.flag))
        print('class_method for self.__y => {0}'.format(cls.__pri_flag))

# 我们可以动态的给实例增加属性的；例如，instance.vars = 5
instance = MyClass(3, 4)

instance.instance_method()
instance.class_method()
```

> Note:
类方法 => （在实例方法上面加上一个 @classmethod 就是类方法了）可以是私有方法，也可以是公有方法；它与普通函数只有一个特别的区别，就是他们必须有一个额外的第一个参数名，但是在调用者哥方法的时候
	  你不能为这个参数赋值，Python会提供这个值，这个特别的变量就是对象本身，按照惯例，它的名字是 self

>	  例如；如果你有一个类，类名叫“MyClass”，当你调用者哥对象的方法 MyObject.method(arg1, arg2)的时候，这会由Python自动转为 
	  MyClass.method(Myobject, arg1, arg2)；

> 类变量 => 被定义在类方法外部的变量的称为类变量，它是共享，所有实例都共享此变量；可以是公有的，也可是私有的

> 实例方法 => 就是类中定义的一个个的函数，这就是实例方法

再看一个静态方法的例子...

```
class MyClass:
    __vars = 5
    @staticmethod
    def get_value():
        return __vars
```

> 静态方法 => （在实例方法上面加上一个 @staticmethod 就是静态方法了）

> 注意：实例方法 和 实例的私有方法 都可以被实例调用；但是实例的私有方法和私有属性是不能在类外部被直接访问使用的；

> 注意：实例方法 的第一个参数是实例的本身self；类方法 的第一个参数是类的本身cls；

> 注意：类方法不能访问实例变量

> 注意：静态方法不能访问类变量

###### 总结
- 类级别：（类和实例都可以访问）
 - 类方法 => 需要传递类本身cls
 - 类变量 => 类定义的时候被确定；

- 实例级别：（只有实例可以访问）
 - 实例方法 => 需要传递实例本身self
 - 实例变量 => 初始化的时候才确定；

- 私有变量与公有变量：
 - 私有：只在类内部可以访问
 - 公有：类外部可以访问，包括类里面

### 继承
