
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
属于类本身这个对象的属性，所有该类的对象都共享类变量。

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
        self.__name = name # 私有属性  
  
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


# 2 继承和混入
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
