---
title: Python编程语言--面向对象
date: 2023-10-16 16:27:41
categories:
  - Go & Python
tags:
  - 程序设计
  - Python
published: true
---

# 1 类
## 1.1 类的定义和使用
```python
 class Dog:
     """一次模拟小狗的简单尝试。"""

     def __init__(self, name, age):
          """初始化属性name和age。"""
         self.name = name
         self.age = age
         self.type = 1  # 给属性指定默认值
          

     def sit(self):
          """模拟小狗收到命令时蹲下。"""
          print(f"{self.name} is now sitting.")

      def roll_over(self):
          """模拟小狗收到命令时打滚。"""
          print(f"{self.name} rolled over!")

my_dog = Dog('Willie', 6)
my_dog.name
my_dog.sit()
my_dog.roll_over()
```

创建实例时，有些属性无须通过形参来定义，可在方法__init__() 中为其指定默认值。

## 1.2 `__init__` 方法
`__init__` 方法是类的特殊方法之一，它用于初始化类的实例。
- 通过默认参数给属性提供默认值

```python
class Person:
    def __init__(self, name="Unknown", age=0):
        self.name = name
        self.age = age

# 创建 Person 类的实例
person1 = Person()
person2 = Person("Bob", 25)
```

- 做其他初始化工作

```python
class FileHandler:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = open(filename, mode)

    def close(self):
        self.file.close()

# 创建 FileHandler 类的实例
file_handler = FileHandler("example.txt", "w")
file_handler.file.write("Hello, World!")
file_handler.close()
```
以上示例中在 `__init__` 方法中进行文件打开。

## 1.3 类变量
属于类本身这个对象的属性，所有该类的对象都共享类变量。类变量可以通过类名、对象名调用。

```python
from pprint import pprint  
  
class Student:  
    student_count = 8  # 类变量  
  
def main():  
    print(Student.__name__)  # Student
  
    print(Student.student_count)  
    print(getattr(Student, "student_count"))  
    # print(Student.unknown)  
    # 获取不存在的类变量，返回默认值  
    print(getattr(Student, "unknown", "10"))  
  
    Student.student_count = 89  
    setattr(Student, "student_count", 100)  
    print(Student.student_count)  
  
    Student.newattribute = "hello"  
    print(Student.newattribute)  
  
    # del Student.newattribute  
    # 删除类变量  
    delattr(Student, "newattribute")  
    # print(Student.newattribute)  
  
    s1 = Student()  
    s2 = Student()  
    Student.student_count = 4  
    print(s1.student_count)  
    print(s2.student_count)  
  
    # 类变量都存储在类的 __dict__ 中  
    pprint(Student.__dict__)  
  
if __name__ == '__main__':  
    main()
```

## 1.4 实例变量与函数
**定义实例变量与函数：**

```python
class Student:  
    def __init__(self, name: str):  
        self.name = name  
  
    def say_hello(self, msg: str):  
        print(f"Hello {msg}, {self.name}")  

def main():  
    # 1. create a physical object  
    # 2. call __init__() to initialize this object    s1 = Student("Jack")  
    s2 = Student("Tom")  
  
    s1.say_hello("1111")  
    s2.say_hello("2222")  
  
    s1.gender = 'Male'  # 类似定义类变量，定义实例变量  
    print(s1.gender)  
    print(s2.gender)  # 报错：s2中没有gender属性
```

## 1.5 私有属性与函数
在Python中并没有严格的权限限定符去进行限制，主要通过命名来进行区分。

```python
class Student:  
    def __init__(self, name: str):  
        self.__name = name # 私有属性（双下划线）
  
    def __say_hello(self, msg: str):  
        print(f"Hello {msg}, {self.__name}")  
  
def main():  
    s1 = Student("Jack")  
    print(s1.__name)  # 报错 无法使用私有属性，需要用下边的方式  
    print(s1._Student__name)   # _classname__attribute  
  
    s1._Student__say_hello("1111")  
  
if __name__ == '__main__':  
    main()
```

## 1.6 `__new__`方法
`__new__` 方法是在对象实例化之前由 Python 解释器调用的，而 `__init__` 方法则在对象实例化之后调用，用于对象的初始化。

可以在你的类中定义 `__new__` 方法，通常会接受类本身作为第一个参数（通常命名为 `cls`），并返回一个对象实例。这个方法的主要目的是创建对象实例。
`__new__` 方法允许你控制对象的创建过程。


```python
class Student:  
    def __new__(cls, first_name, last_name):  
        obj = super().__new__(cls)  
        obj.first_name = first_name  
        obj.last_name = last_name  
        return obj

student = Student("Jack", "Ma")  
print(student.first_name)  
print(student.last_name)
```


# 2 继承
## 2.1 方法重载
子类通过重写父类的方法，实现方法的重载：

```python
class Person:  
    color = 1  
  
    def __init__(self):  
        self.name = "Jack"  
  
    def say(self):  
        print("Hello from person")  
  
    def print_color(self):  
        print(self.color)  
  
  
class Student(Person):  
    color = 2  # 重载类属性
  
    def __init__(self):  
        super().__init__()  
        self.school = "Abc"  
  
    def say(self):   # 重载父类方法
        super().say()  
        print("Hello from student")

def render(person: Person):  # 可以传person及其子类的对象
    person.say()
```
子类可以重载父类的方法和类属性。

## 2.2 继承和混入
- **子类调用父类使用 `super()`**

```python
class Father(object):
    def run(self):
        print("run in father")

class Son(Father):
    def run(self):
        super().run()
        print("run in son")

obj2 = Son()  
obj2.run()
```
输出：
```shell
run in father
run in son
```

- **多继承和混入**

```python
import json

class JSONMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class JSONPerson(Person, JSONMixin):
    pass

person = JSONPerson("Alice", 30)
json_data = person.to_json()
print(json_data)  # 输出 '{"name": "Alice", "age": 30}'
```
在上面的示例中，`JSONPerson` 类继承了 `Person` 类和 `JSONMixin` 类。通过混入，`JSONPerson` 类获得了 `to_json` 方法，可以将其属性转换为 JSON 字符串。

> 1. 多继承可以在某些情况下提供灵活性，但要小心避免菱形继承问题（即多个父类共同继承同一个祖先类，可能导致方法冲突）。
> 2. 混入通常用于将通用功能添加到多个类中，提供代码重用和模块化。混入类通常不应该实例化，而应该与其他类一起使用。


## 2.3 抽象类
抽象类是一个不能被实例化的类，抽象方法是一个没有具体实现的方法，Python并没有直接支持抽象类，提供了一个模块（abc）来允许定义抽象类。
抽象类的定义：

```python
from abc import ABC, abstractmethod  
  
class Action(ABC):  
    @abstractmethod  
    def execute(self):  
        pass  
  
  
class CreateStudentAction(Action):  
    def execute(self):  # 子类必须实现抽象类的所有抽象方法
        print("Create a new student")  
  
  
def execute_action(action: Action):  
    action.execute()  
  
  
def main():  
    create_student_action = CreateStudentAction()  
    # delete_student_action = DeleteStudentAction()  
  
    execute_action(create_student_action)  # Create a new student
    # execute_action(delete_student_action)  
  
  
if __name__ == '__main__':  
    main()
```

抽象类的主要用途是定义一组接口或方法，以确保派生类提供了必需的实现。一般的使用场景包括：
1. **制定接口规范**：抽象类可以定义一组方法，然后子类需要提供这些方法的具体实现。这对于确保多个类遵循相同的接口规范非常有用。
2. **强制实现**：通过使用抽象类，你可以确保派生类提供了必要的实现，而不会遗漏任何关键方法。
3. **代码结构组织**：抽象类有助于组织代码结构，将通用的接口和方法集中到基类中，减少了代码重复。
4. **多态性**：抽象类可以用于多态性，允许不同的子类实现相同的接口，这有助于更容易地切换或扩展功能。

## 3.3 多继承
1. 子类继承多个父类，同名方法存在于多个父类，按照继承顺序优先调用第一个继承的父类方法
2. 通过类名指定调用那个父类的方法，需要传self参数

```python
class Parent1:  
    def __init__(self):  
        pass  
  
    def render(self):  
        print("parent 1")  
  
    def hello(self):  
        print("hello parent 1")  
  
  
class Parent2:  
    def __init__(self):  
        pass  
  
    def render(self):  
        print("parent 2")  
  
    def hello(self):  
        print("hello parent 2")  
  
  
class Child(Parent2, Parent1):  
    def __init__(self):  
        Parent1.__init__(self)  
  
    def render(self):  
        super().render()  # 默认调用 Parent2  
    def hello(self):  
        Parent1.hello(self) # 指定调用的父类，需要传参self  
  
  
def main():  
    child = Child()  
    child.render()  # parent 2
    child.hello()   # hello parent 1
  
if __name__ == '__main__':  
    main()
```


# 3 枚举
## 3.1 枚举的定义
Python提供了一个名为`enum`的模块，用于创建和使用枚举。定义枚举类型：

```python
class Gender(Enum):  
    MALE = 1  
    FEMALE = 2

class Student:  
    def __init__(self, gender: Gender):  
        self.gender = gender

def main():  
    print(type(Gender.MALE))  # <enum 'Gender'>
    print(Gender.MALE.name)  # MALE
    print(Gender.MALE.value) # 1

	for gg in Gender:  
	    print(gg)  # 迭代输出枚举对象

	student = Student(Gender.MALE)  
	if student.gender == Gender.MALE:  
	    print("This student is a male")  
	else:  
	    print("This student is a female")  
	  
	s_gender = "FEMALE"  
	student.gender = Gender[s_gender] # 可以使用key值，加方括号，取对应的枚举对象  
	print(student.gender)  # Gender.FEMALE  
	  
	i_gender = 2  
	print(Gender(i_gender))  # 可以使用value值，加小括号，取对应的枚举对象
```
枚举类型中存储的格式为key-value，枚举类型可以进行迭代。

定义枚举时，可以使用`auto()`让Python自动分配值：

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()    # 1
    GREEN = auto()  # 2
    BLUE = auto()   # 3

print(Color.BLUE.value) # 3
```


## 3.2 枚举别名和装饰器
如果定义的枚举中有相同的value值，则称为同一个值的枚举别名，底层其实是同一个枚举。

```python
from enum import Enum  
  
  
class Status(Enum):  
    SUCCESS = 1  
    OK = 1  
    FAIL = 2  
    WRONG = 2

def main():  
    for s in Status:  
        print(s.name)  # 仅输出 SUCCESS FAIL
  
    print(Status.__members__)  
    
    print(Status.SUCCESS == Status.OK) 
    print(Status.SUCCESS is Status.OK) 
    
if __name__ == '__main__':  
    main()
```
输出：
```shell
SUCCESS
FAIL
{'SUCCESS': <Status.SUCCESS: 1>, 'OK': <Status.SUCCESS: 1>, 'FAIL': <Status.FAIL: 2>, 'WRONG': <Status.FAIL: 2>}
True
True
```

通过使用枚举装饰器 `@enum.unique`可以禁止定义枚举别名，防止同一个value定义不同的枚举，造成冲突。

```python
import enum

@enum.unique  
class Gender(Enum):  
    MALE = 1  
    # OK = 1 # 报错：ValueError: duplicate values found in <enum 'Gender'>: OK -> MALE
    FEMALE = 2
```

## 3.3 定制和扩展枚举
通过重写枚举类中的 `__str__`、`__eq__`等方法，实现不同的功能：

```python
import enum  
from enum import Enum  
  
@enum.unique  
class Gender(Enum):  
    MALE = 1  
    FEMALE = 2  
  
    def __str__(self):  
        return f"{self.name}({self.value})"  
  
    def __eq__(self, other):  
        if isinstance(other, int):  
            return self.value == other  
  
        if isinstance(other, str):  
            return self.name == other.upper()  
  
        if isinstance(other, Gender):  
            return self is other  
  
        return False

for g in Gender:  
    print(g)  
  
print(Gender.MALE == 1)  # True
print(Gender.MALE == "FEMALE") # False
```

如果定义比较相关操作，可以使用 `total_ordering` 装饰器，你只需要定义一个或多个比较操作方法中的一个（例如 `__lt__`、`__le__`、`__eq__`、`__ne__`、`__gt__` 或 `__ge__`），然后装饰器将自动为你生成其余的方法。

```python
from functools import total_ordering

@total_ordering
class MyClass:
    def __init__(self, value):
        self.value = value

    def __eq__(self, other):
        return self.value == other.value

    def __lt__(self, other):
        return self.value < other.value

# 创建两个对象
obj1 = MyClass(5)
obj2 = MyClass(10)

# 使用比较操作符进行比较
print(obj1 < obj2)  # 输出: True
print(obj1 == obj2)  # 输出: False
print(obj1 <= obj2)  # 输出: True
print(obj1 > obj2)  # 输出: False
print(obj1 >= obj2)  # 输出: False
```

# 4 描述符

Python中的描述符（descriptor）是一种强大的编程机制，允许你在对象属性的访问和修改过程中定义自定义行为。描述符是通过实现特定方法的类来实现的，这些方法包括 `__get__`、`__set__` 和 `__delete__`。描述符通常用于控制属性的访问和修改，以便实现数据封装和验证。

下面是描述符的主要功能和用法：

1. **`__get__` 方法**：当你尝试访问属性时，`__get__` 方法会被调用。这允许你自定义属性的获取行为。通常，`__get__` 方法返回属性的值。
2. **`__set__` 方法**：当你尝试为属性分配一个新值时，`__set__` 方法会被调用。这允许你自定义属性的设置行为。你可以在 `__set__` 方法中进行验证和限制。
3. **`__delete__` 方法**：当你尝试删除属性时，`__delete__` 方法会被调用。这允许你自定义属性的删除行为。通常，`__delete__` 方法用于清理或处理与属性相关的资源。


```python
class CustomDescriptor:
    def __get__(self, instance, owner):
        print("Getting the value")
        return instance._value

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Value cannot be negative")
        print("Setting the value")
        instance._value = value

    def __delete__(self, instance):
        print("Deleting the value")
        del instance._value

class MyClass:
    def __init__(self, value):
        self._value = value
    descriptor = CustomDescriptor()

obj = MyClass(10)
print(obj.descriptor)  # 访问属性，触发 __get__ 方法
obj.descriptor = 20   # 设置属性，触发 __set__ 方法
del obj.descriptor     # 删除属性，触发 __delete__ 方法

```
输出：
```shell
Getting the value
10
Setting the value
Deleting the value
```

在上面的示例中，`CustomDescriptor` 类实现了描述符的三个主要方法。`MyClass` 类中的 `descriptor` 属性使用了这个自定义描述符。当我们访问、设置和删除这个属性时，对应的 `__get__`、`__set__` 和 `__delete__` 方法被调用，这使得我们可以自定义属性的行为。

描述符通常用于以下场景：

- **属性验证和控制**：描述符允许你在属性赋值时进行验证，确保属性值满足特定条件，如范围限制、数据类型等。
- **属性访问控制**：你可以使用描述符来控制属性的访问权限，例如将属性设置为只读或私有。
- **属性懒加载**：描述符可以用于延迟加载属性的值，直到首次访问。
- **属性计算**：你可以使用描述符来计算属性的值，而不是存储它们。