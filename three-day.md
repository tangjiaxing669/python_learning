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

我们再来验证下，看个例子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class Class_A(object):
    public_class_a_var = 'public class a var'
    __pri_class_a_var = 'private class a var'

    def __init__(self):
        self.public_instance_a_var = 'public instance a var'
        self.__pri_instalce_a_var = 'private instance a var'

    def public_instance_a(self):
        try:
            print(self.public_instance_a_var)
        except:
            pass
        try:
            print(self.__pri_instalce_a_var)
        except:
            pass
        try:
            print(self.public_class_a_var)
        except:
            pass
        try:
            print(self.__pri_class_a_var)
        except:
            pass

    def __pri_instance_a(self):
        try:
            print(self.public_instance_a_var)
        except:
            pass
        try:
            print(self.__pri_instalce_a_var)
        except:
            pass
        try:
            print(self.public_class_a_var)
        except:
            pass
        try:
            print(self.__pri_class_a_var)
        except:
            pass

    @classmethod
    def public_class_method(cls):
        try:
            print(cls.public_class_a_var)
        except:
            pass
        try:
            print(cls.__pri_class_a_var)
        except:
            pass
        try:
            print(cls.public_instance_a_var)
        except:
            pass
        try:
            print(cls.__pri_instalce_a_var)
        except:
            pass

    @classmethod
    def __pri_class_method(cls):
        try:
            print(cls.public_class_a_var)
        except:
            pass
        try:
            print(cls.__pri_class_a_var)
        except:
            pass
        try:
            print(cls.public_instance_a_var)
        except:
            pass
        try:
            print(cls.__pri_instalce_a_var)
        except:
            pass

a1 = Class_A()
```
当我们依次执行下面的操作时...

```
执行 a1.public_instance_a()
    输出：
	public instance a var
	private instance a var
	public class a var
	private class a var
	
执行 a1.__pri_instance_a()
    输出：
	Traceback (most recent call last):
	  File "/root/python3/OOP_3.py", line 91, in <module>
	    a1.__pri_instance_a()
	AttributeError: 'Class_A' object has no attribute '__pri_instance_a'
	
执行 a1.public_class_method()
    输出：
	public class a var
	private class a var
	
执行 a1.__pri_class_method()
    输出：
	Traceback (most recent call last):
	  File "/root/python3/OOP_3.py", line 93, in <module>
	    a1.__pri_class_method()
	AttributeError: 'Class_A' object has no attribute '__pri_class_method'
	
执行 print(a1.public_class_a_var)
    输出：
	public class a var
    	
执行 print(a1.__pri_class_a_var)
    输出：
	Traceback (most recent call last):
	  File "/root/python3/OOP_3.py", line 95, in <module>
	    print(a1.__pri_class_a_var)
	AttributeError: 'Class_A' object has no attribute '__pri_class_a_var'
	
执行 print(a1.public_instance_a_var)
    输出：
	public instance a var
    	
执行 print(a1.__pri_instance_a_var)
    输出：
	Traceback (most recent call last):
	  File "/root/python3/OOP_3.py", line 97, in <module>
	    print(a1.__pri_instance_a_var)
	AttributeError: 'Class_A' object has no attribute '__pri_instance_a_var'
```

所以，我们再次总结...
* 共有实例方法可以访问共有和私有类变量，以及共有和私有实例变量
* 私有方法和私有变量在类的外部都不能被访问
* 共有方法和共有的变量在类外部都可以被访问
* 共有类方法只能访问共有和私有的类变量

### 继承

> 继承分为但继承和多继承；我们先看一个单继承的例子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class Door(object):
    def __init__(self, number, status):
        self.number = number
        self.status = status

    def open_door(self):
        if self.status == 'closed':
            self.status = 'open'

    def close_door(self):
        self.status = 'closed'

class Lock_Door(Door):
    def __init__(self, number, status, lock_status):
        super(Lock_Door, self).__init__(number, status)
        self.lock_status = lock_status

    def open(self):
        if self.status == 'closed':
            super(Lock_Door, self).open_door()
        else:
            print('The Door is Locked.')

    def lock(self):
        if self.status == 'closed':
            self.lock_status = True
        else:
            print('The Door is opening.')

door = Lock_Door('123', 'open', False)

door.lock()
```

> 在上面的例子中，我们定义了一个`Door`类，它有两个属性，`number`和`status`，分别指**门牌号**和**门状态**，**门状态**仅有开和关两种状态；然后分别定义了两个实例方法，`open_door`和`close_door`；我们还定义了一个`Lock_Door`的类，它继承了`Door`这个类，并且自己还拥有一个独立的属性`lock_status`，代表门有没有被上锁，然后还拥有一个`lock`的方法；*（`super`语句仅用来初始化父类的；上面的`super(Lock_Door, self).__init__(number, status)`代表初始化父类`Door`；`super(Lock_Door, self).open_door()`代表调用父类的`open_door()`方法）*

现在再来看一个多继承的例子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class A:
    def method_from_a(self):
        print('Method From A')

class B:
    def method_from_b(self):
        print('Method From B')

class C(A, B):
    pass

c = C()
c.method_from_a()
c.method_from_b()
```

> 上面我们定义了三个类`A`、`B`、`C`；类`C`依次继承了类`A`和类`B`，我们实例化类`C`后，同时获得了类`A`和类`B`中的所有属性方法；

> 我们再看一个例子，将上面的代码该一改....

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class A:
    def method(self):
        print('method from a')

class B:
    def method(self):
        print('method from b')

class C(A, B):
    pass

c = C()
c.method()
```

执行结果如下：

```
method from a
```

> 上面的代码中，当我们将`class C(A, B)`改为`class C(B, A)`时，执行结果为`method from b`；所以由此得出，多继承中，继承的顺序时从左至右的......

现在我们将上面的代码再改成如下：

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class A:
    def method(self):
        print('method from a')

class B(A):
    def method(self):
        print('method from b')

class C(A, B):
    pass

c = C()
c.method()
```

> 上面我们定义了三个类`A`、`B`、`C`；类A中拥有`method`方法，类`B`中也拥有`method`方法，并且类`B`是继承至类`A`；然后我们定类`C`，依次继承类`A`和类`B`，我们再实例化类`C`，按理说类`C`实例化后会获得类`A`和类`B`中所有的属性和方法，当我们调用`c.method()`实例方法时，结果却抛出了异常，如下：

```
Traceback (most recent call last):
  File "/root/python3/OOP_5.py", line 13, in <module>
    class C(A, B):
TypeError: Cannot create a consistent method resolution
order (MRO) for bases A, B
```

> 这是为啥？异常说我们的`class C(A, B)`这里有问题，那么我们试着将`A`和`B`换个位置...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class A:
    def method(self):
        print('method from a')

class B(A):
    def method(self):
        print('method from b')

class C(B, A):
    pass

c = C()
c.method()
```

执行结果如下：

```
method from b
```

> God.....

这里就要涉及到`C3`算法了，我们先看下`C3`算法的`MERGE`规则....
> 这个只针对`Python`...

C3算法的MERGE步骤
* 顺序遍历列表
* 首元素满足以下调教，否则遍历下一个序列
 - 在其他序列也是首元素
 - 在其他序列里不存在
* 从所有序列中移除此元素，合并到MRO序列中
* 重复执行，直到所有序列为空或无法执行下去

```
	C(A, B) -> C 继承 A，也继承 B；下面的 O 是指默认继承的Object对象
	[C] + merge(MRO(A), MRO(B), [A, B])
	[C] + merge([A, O], [B, O], [A, B])
	[C, A] + merge([O], [B, O], [B])
	[C, A, B] + merge([O], [O])
	[C, A, B, O]	所以 C 会依次继承 A B O

	C(B, A) -> C 继承 B，再继承 A；下面的 O 是指默认继承的Object对象
	[C] + merge(MRO(B), MRO(A), [B, A])
	[C] + merge([B, O], [A, O], [B, A])
	[C, B] + merge([O], [A, O], [A])
	[C, B, A] + merge([O], [O])
	[C, B, A, O]	所以 C 会依次继承 B A O

	C(A, B), B(A) -> B 继承 A，C 先继承 A，再继承 B
	我们可以先算出 B(A)
	[B] + merge(MRO(A), [A])
	[B] + merge([A, O], [A])
	[B, A] + merge([O])
	[B, A, O]
	再算出 C(A, B)
	[C] + merge(MRO(A), MRO(B), [A, B])
	[C] + merge([A, O], [B, A, O], [A, B])
	注意：MRO(B) 就等于上面算出来的 [B, A, O]
	注意：这里的 A 在三个序列中都出现，但不是首元素，B 也一样
	      所以这里的继承会抛出异常，
	      raise TypeError

	      Traceback (most recent call last):
	        File "/root/python3/OOP_3.py", line 13, in <module>
	          class Class_C(Class_A, Class_B):
	      TypeError: Cannot create a consistent method resolution
	      order (MRO) for bases Class_A, Class_B
	
	C(B, A), B(A) -> B 继承 A，C 先继承 B，再继承 A
	[C] + merge(([B] + merge(MRO(A), [A])), MRO(A), [B, A])
	[C] + merge(([B] + merge([A, O], [A])), [A, O], [B, A])
	[C] + merge(([B, A], merge([O])), [A, O], [B, A])
	[C] + merge([B, A, O], [A, O], [B, A])
	[C, B] + merge([A, O], [A, O], [A])
	[C, B, A] + merge([O], [O])
	[C, B, A, O]
	注意：上面的 B(A) 得出下面的 [B, A, O]
	所以这里的继承关系是正确的，不会抛出异常
```

### 对象的创建与销毁

- `__new__`创建对象
- `__init__`初始化对象
- `__del__`销毁对象的时候调用

如下代码：

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class Test1:
    def __new__(cls):
        print('call __new__')
        return object.__new__(cls)

    def __init__(self):
        print('call __init___')

    def method(self):
        print('call method')

    def __del__(self):
        print('call __del__')

c = Test1()
```

当运行以上代码时，输出如下：

```
call __new__
call __init___
call __del__
```

> 也就是当你定义了`__new__`、`__init__`、`__del__`这三种方法时，程序默认都会执行依次执行，程序的普通方法还是正常使用；而当你使用`del c`来删除这个实例的时候，程序才会调用`__del__`方法.

### 可视化对象

我们再来看看类中`__repr__`、`__str__`和`__bytes__`的作用...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class Test2:
    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return self.name

a = Test2('Jason tom')
print(a)
```

默认情况下，当我们实例化`a = Test2`时，`a`已经被实例化为一个对象了，此时你要执行`print(a)`会得到一个类似`<__main__.a instance at 0x2395518>`这样的值，如果你想改变这个值，那么`__repr__`可以帮到你；现在我们执行上面的代码...

```
Jason tom
```

其结果将会变成上面的那样；现在我们将上面的代码改成如下：

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

class Test2:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return 'call __str__ name is {0}'.format(self.name)

    def __bytes__(self):
        return 'cal __bytes__ name is {0}'.format(self.name).encode('utf-8')

a = Test2('Jason tom')
print(str(a))
print(bytes(a))
```

> 注意，上面的`__str__`返回的是一个`str`类型的值，`__bytes__`返回的是`bytes`类型的值(需要解码)；

### 比较运算符重载

当我们想要对一个实例化的对象进行比较的时候，那么下面的代码可以帮到你...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"


class Person:
    def __init__(self, age):
        self.age = age

    def __lt__(self, other):
        print('lt')
        return self.age < other.age

    def __le__(self, other):
        print('le')
        return self.age <= other.age

    def __eq__(self, other):
        print('eq')
        return self.age == other.age

    def __ne__(self, other):
        print('ne')
        return self.age != other.age

    def __gt__(self, other):
        print('gt')
        return self.age > other.age

    def __ge__(self, other):
        print('ge')
        return self.age >= other.age

a = Person(18)
b = Person(20)

print(a > b)
print(a < b)
```

执行结果如下：

```
gt
False
lt
True
```
