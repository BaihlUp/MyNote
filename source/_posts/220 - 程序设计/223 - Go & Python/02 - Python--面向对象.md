
# 6 类
## 6.1 类的定义和使用
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

## 6.2 `__init__` 方法
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

## 6.2 继承和混入
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
