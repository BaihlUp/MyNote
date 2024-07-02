---
title: Java--面向对象编程
date: 2024-06-30
categories:
  - Java & Lua
tags:
  - 程序设计
  - Java
published: false
---
# 1 类和对象
## 1.1 类的定义

1. 是用来描述同一类事物的
2. 可以在内部定义任意数量的、不同类型的变量，作为这一类事物的属性。这种属性叫做成员变量 ( member variable )。
3. 如果一个Java文件中定义了一个public类，则文件名必须与该类的名称相同，包括大小写。
4. 如果一个Java文件中定义了多个类，则只能有一个public类，该类的名称必须与文件名相同。
5. 就好像文件路径+文件名不能重复一样，一个Java程序中相同名字的类只能有一个

```java
// >> TODO 一个类以public class开头，public class代表这个类是公共类，类名必须和文件名相同。
// >> TODO public class后面紧跟类名，然后是一对打括号的类体
public class Merchandise {
    // >> TODO 类体中可以定义描述这个类的属性的变量。我们称之为成员变量（member variable）
    // >> TODO 每个成员变量的定义以;结束
    String name;
    String id;
    int count;
    double price;

}

// >> TODO 上面这整个类，其实就是创建了一个模版。描述了一种我们需要的数据类型。
```

## 1.2 匿名对象
可以不定义对象的句柄，而直接调用这个对象的方法，如下：`new Person().shout();`
使用情况：
- 如果一个对象只需要执行一次方法调用，那么就可以使用匿名对象
- 将匿名对象作为实参传递给一个方法调用
## 1.3 引用类型
Java中的数据类型分为基本数据类型和引用数据类型

- **引用数据类型和基本数据类型的相同点**

1. 都可以用来创建变量，可以赋值和使用其值
2. 本身都是一个地址

- **引用数据类型和基本数据类型的不同点**

1. 基本类型变量的值，就是地址对应的值。引用数据类型的值还是一个地址，需要通过“二级跳”找到实例
2. 引用数据类型是Java的一种内部类型，是对所有自定义类型和数组引用的统称，并非特指某种类型

```java
public class Merchandise {
    String name;
    String id;
    int count;
    double price;

}
public class Merchandise {
    String name;
    String id;
    int count;
    double price;
}

public class ReferenceAndPrimaryDataType {
    public static void main(String[] args) {

        // >> TODO m1是一个Merchandise类型的引用，只能指向Merchandise类型的实例
        // >> TODO 引用数据类型变量包含两部分信息：类型和实例。也就是说，
        //    TODO 每一个引用数据类型的变量（简称引用），都是指向某个类（ class /自定义类型）
        //    TODO 的一个实例/对象（instance / object）。不同类型的引用在Java的世界里都是引用。
        // >> TODO 引用的类型信息在创建时就已经确定，可以通过给引用赋值，让其指向不同的实例.
        //         比如 m1 就是Merchandise类型，只能指向Merchandise的实例。
        Merchandise m1;
        m1 = new Merchandise();
        Merchandise m2 = new Merchandise();
        Merchandise m3 = new Merchandise();
        Merchandise m4 = new Merchandise();
        Merchandise m5 = new Merchandise();

        // >> TODO 给一个引用赋值，则两者的类型必须一样。m5可以给m1赋值，因为他们类型是一样的
        m1 = m5;

        System.out.println("m1=" + m1);
        System.out.println("m2=" + m2);
        System.out.println("m3=" + m3);
        System.out.println("m4=" + m4);
        System.out.println("m5=" + m5);

        Merchandise m6 = m1;
        System.out.println("m6=" + m6);
        m6 = m5;
        System.out.println("m6=" + m6);


        System.out.println("m1=" + m1);
        System.out.println("m2=" + m2);
        System.out.println("m3=" + m3);
        System.out.println("m4=" + m4);
        System.out.println("m5=" + m5);


        int a = 999;

    }
}

```
输出：
```
m1=Merchandise@1b6d3586
m2=Merchandise@4554617c
m3=Merchandise@74a14482
m4=Merchandise@1540e19d
m5=Merchandise@1b6d3586
m6=Merchandise@1b6d3586
m6=Merchandise@1b6d3586
m1=Merchandise@1b6d3586
m2=Merchandise@4554617c
m3=Merchandise@74a14482
m4=Merchandise@1540e19d
m5=Merchandise@1b6d3586
```
以上的m1 == m5 == m6，引用的地址一样。

## 1.4 类对象和引用的关系
- **类和对象的关系**

1. 类是对象的模版，对象是类的一个实例
2. 一个Java程序中类名相同的类只能有一个，也就是类型不会重名
3. 一个类可以有很多对象
4. 一个对象只能根据一个类来创建

- **引用和类以及对象的关系**

1. 引用必须是、只能是一个类的引用
2. 引用只能指向其所属的类型的类的对象
3. 相同类型的引用之间可以赋值
4. 只能通过指向一个对象的引用，来操作一个对象，比如访问某个成员变量

- **数组是一种特殊的类**

1. 数组的类名就是类型带上中括号
2. 同一类型的数组，每个数组对象的大小可以不一样，也就是**每个数组对象占用的内存可以不一样**，这点和类的对象不同。
3. 可以用引用指向类型相同大小不同的数组，因为他们属于同一种类型

- **引用数组**

1. 可以把类名当成自定义类型，定义引用的数组，甚至多维数组

- 示例1
```java
public class ArrayIsClass {
    public static void main(String[] args) {
        // >> TODO “数组变量”其背后真身就是引用。数组类型就是一种特殊的类。
        // >> TODO 数组的大小不决定数组的类型，数组的类型是只是由元素类型决定的。
        int[] intArr;
        intArr = new int[1];
        intArr = new int[2];

        // 这个数组的元素就是二维的double数组，既double[][]
        double[][][] double3DArray = new double[2][3][4];

        int[] a1 = new int[9];
        int[] a2 = new int[0];
        a2 = a1;
        System.out.println("a2.length=" + a2.length);
        double[] a3 = new double[5];
        // a3是double[]类型的引用，不可以用int[]类型的引用赋值。
        double3DArray[1][2] = a3;
    }
}
```
输出：
```
a2.length=9
```

- 示例2

```java
public class RefArray {

    public static void main(String[] args) {
        Merchandise[] merchandises = new Merchandise[9];
        merchandises[0] = new Merchandise();
        merchandises[1] = new Merchandise();
        merchandises[0].name = "笔记本";
        System.out.println(merchandises[0].name);

        System.out.println(merchandises[2]);
    }
}

```
输出:
```
笔记本
null
```

## 1.5 对象的内存解析
### 1.5.1 JVM内存结构划分
HotSpot Java虚拟机的架构图如下。其中主要关心的是运行时数据区部分（Runtime Data Area）。
![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240701182618.png)

堆：此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。
栈：是指虚拟机栈。虚拟机栈用于存储局部变量等。
方法区：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

### 1.5.2 对象内存解析
示例：
```java
class Person { //类:人 String name;
int age;
    boolean isMale;
}

public class PersonTest { //测试类
	public static void main(String[] args) {
		Person p1 = new Person(); p1.name = "赵同学";
		p1.age = 20;
		p1.isMale = true;
		
		Person p2 = new Person();
		p2.age = 10;
		
		Person p3 = p1;
		p3.name = "郭同学"; 
	}
}
```
内存解析图：
![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240701183147.png)

- 堆：凡是new出来的结构（对象、数组）都放在堆空间中。
- 对象的属性存放在堆空间中
- 创建一个类的多个对象（比如：p1、p2），则每个对象都拥有当前类的一套“副本”（即属性）。当通过一个对象修改其属性时，不会影响其他对象此属性的值
- 当声明一个新的变量使用现有的对象进行赋值时（比如p3=p1），两个变量会共同指向堆空间中同一个对象


## 1.6 类方法

### 1.6.1 构造方法
1. 构造方法（constructor）的方法名必须与类名一样，而且构造方法没有返回值。这样的方法才是构造方法。
2. 构造方法可以有参数，规则和语法于普通方法一样。使用时，参数传递给 new 语句后类名的括号后面。
3. 如果没有显示的添加一个构造方法，Java会给每个类都会默认自带一个无参数的构造方法。
4. 如果我们自己添加类构造方法，Java就不会再添加无参数的构造方法。这时候，就不能直接 new 一个对象不传递参数了（看例子）
5. 所以我们一直都在使用构造方法，这也是为什么创建对象的时候类名后面要有一个括号的原因。
6. 构造方法无法被点操作符调用或者在普通方法里调用，只能通过 new 语句在创建对象的时候，间接调用。
7. 理解一下为什么构造方法不能有返回值，因为有返回值也没有意义，new 语句永远返回的是创建出来的对象的引用

```java
package com.geekbang.supermarket;

public class MerchandiseV2 {

    public String name;
    public String id;
    // >> TODO 构造方法执行前，会执行给局部变量赋初始值的操作
    // >> TODO 我们说过，所有的代码都必须在方法里，那么这种给成员变赋初始值的代码在哪个方法里？怎么看不到呢？
    //    TODO 原来构造方法在内部变成了<init>方法。学习就是要脑洞大，敢想敢试，刨根问底。
    public int count = 999;// 999/0;  在构造方法前赋值变量
    public double soldPrice;
    public double purchasePrice;

    // >> TODO 构造方法（constructor）的重载和普通方法一样
    public MerchandiseV2(String name, String id, int count, double soldPrice, double purchasePrice) {
        this.name = name;
        this.id = id;
        this.count = count;
        this.soldPrice = soldPrice;
        this.purchasePrice = purchasePrice;
//        soldPrice = 9/0;
    }

    // >> TODO 在构造方法里才能调用重载的构造方法。语法为this(实参列表)
    // >> TODO 构造方法不能自己调用自己，这会是一个死循环
    // >> TODO 在调用重载的构造方法时，不可以使用成员变量。因为用语意上讲，这个对象还没有被初始化完成，处于中间状态。
    // >> TODO 在构造方法里才能调用重载的构造方法时，必须是方法的第一行。后面可以继续有代码
    public MerchandiseV2(String name, String id, int count, double soldPrice) {
         // double purPrice = soldPrice * 0.8;
        // this(name, id, count, soldPrice, purchasePrice);
        this(name, id, count, soldPrice, soldPrice * 0.8);
        // double purPrice = soldPrice * 0.8;
    }

    // >> TODO 因为我们添加了构造方法之后，Java就不会再添加无参数的构造方法。如果需要的话，我们可以自己添加这样的构造方法
    public MerchandiseV2() {
        this("无名", "000", 0, 1, 1.1);

    }

    public void describe() {
        System.out.println("商品名字叫做" + name + "，id是" + id + "。 商品售价是" + soldPrice
            + "。商品进价是" + purchasePrice + "。商品库存量是" + count +
            "。销售一个的毛利润是" + (soldPrice - purchasePrice));
    }

    public double buy(int count) {
        if (this.count < count) {
            return -1;
        }
        return this.count -= count;
    }
}

```

* 在构造方法里才能调用重载的构造方法。语法为this(实参列表)
* 构造方法不能自己调用自己，这会是一个死循环
* 在调用重载的构造方法时，不可以使用成员变量。因为用语意上讲，这个对象还没有被初始化完成，处于中间状态。
* 在构造方法里才能调用重载的构造方法时，必须是方法的第一行。后面可以继续有代码

### 1.6.2 静态变量和静态方法
1. 静态变量

* 静态变量使用 static 修饰符
* 静态变量如果不赋值，Java也会给它赋以其类型的初始值
* 静态变量一般使用全大写字母加下划线分割。这是一个习惯用法
* 所有的代码都可以使用静态变量，只要根据防范控制符的规范，这个静态变量对其可见即可
* public 的静态变量，所有的代码都可以使用它
* 如果没有public修饰符，只能当前包的代码能使用它

```java
public static double DISCOUNT_FOR_VIP = 0.95;
static int STATIC_VARIABLE_CURR_PACKAGE_ONLY = 100;
```

2. 静态方法

* 静态方法中不能使用this自引用，其他和成员方法一样。
* 静态方法不属于某个实例，直接使用类名调用，所以静态方法中不能直接访问成员变量。
* 在静态方法里边，可以自己创建对象，或者通过参数，获取对象的引用调用方法和成员变量。
* 在当前类中访问静态方法可以省略类名
* 使用import static引入一个静态方法或静态变量。
* 静态方法的重载也是一样的，方法签名不同即可：方法名+参数类型
* 判断调用哪个方法，也是根据调用时参数匹配决定的。

**示例：**
```java
package com.geekbang;

import com.geekbang.supermarket.MerchandiseV2;
import  static com.geekbang.supermarket.MerchandiseV2.getVIPDiscount;

public class MerchandiseV2DescAppMain {
    public static void main(String[] args) {
        MerchandiseV2 merchandise = new MerchandiseV2
            ("书桌", "DESK9527", 40, 999.9, 500);

        merchandise.describe();

        // >> TODO 使用import static来引入一个静态方法，就可以直接用静态变量名访问了
        //    TODO import static也可以使用通配符*来引入一个类里所有静态变量
        System.out.println(getVIPDiscount());

    }
}
```

```java
package com.geekbang.supermarket;

public class MerchandiseV2 {

    public String name;
    public String id;
    public int count;
    public double soldPrice;
    public double purchasePrice;

    // >> TODO 静态变量使用 static 修饰符
    public static double DISCOUNT_FOR_VIP = 0.95;

    // >> TODO 静态方法使用static修饰符。
    // 静态方法的方法名没有约定俗称全大写
    public static double getVIPDiscount() {
        // >> TODO 静态方法可以访问静态变量，包括自己类的静态变量和在访问控制符允许的别的类的静态变量
        return DISCOUNT_FOR_VIP;
    }

    // >> TODO 除了没有this，静态方法的定义和成员方法一样，也有方法名，返回值和参数
    // >> TODO 静态方法没有this自引用，它不属于某个实例，调用的时候也无需引用，直接用类名调用，所以它也不能直接访问成员变量
    // >> TODO 当然在静态方法里面，也可以自己创建对象，或者通过参数，获得对象的引用，进而调用方法和访问成员变量
    // >> TODO 静态方法只是没有this自引用的方法而已。
    public static double getDiscountOnDiscount(LittleSuperMarket littleSuperMarket) {
        double activityDiscount = littleSuperMarket.activityDiscount;
        return DISCOUNT_FOR_VIP * activityDiscount;
    }

    public MerchandiseV2(String name, String id, int count, double soldPrice, double purchasePrice) {
        this.name = name;
        this.id = id;
        this.count = count;
        this.soldPrice = soldPrice;
        this.purchasePrice = purchasePrice;
        // soldPrice = 9/0;
    }

    public String getName() {
        return name;
    }

    public MerchandiseV2(String name, String id, int count, double soldPrice) {
        // double purPrice = soldPrice * 0.8;
        // this(name, id, count, soldPrice, purchasePrice);
        this(name, id, count, soldPrice, soldPrice * 0.8);
        // double purPrice = soldPrice * 0.8;
    }

    public MerchandiseV2() {
        this("无名", "000", 0, 1, 1.1);

    }

    public void describe() {
        System.out.println("商品名字叫做" + name + "，id是" + id + "。 商品售价是" + soldPrice
            + "。商品进价是" + purchasePrice + "。商品库存量是" + count +
            "。销售一个的毛利润是" + (soldPrice - purchasePrice) + "。折扣为" + DISCOUNT_FOR_VIP);
    }

    public double calculateProfit() {
        double profit = soldPrice - purchasePrice;
//        if(profit <= 0){
//            return 0;
//        }
        return profit;
    }


    public double buy() {
        return buy(1);
    }

    public double buy(int count) {
        return buy(count, false);
    }

    public double buy(int count, boolean isVIP) {
        if (this.count < count) {
            return -1;
        }
        this.count -= count;
        double totalCost = count * soldPrice;
        if (isVIP) {
            // >> TODO 静态方法的访问和静态变量一样，可以带上类名，当前类可以省略类名
            return totalCost * getVIPDiscount();
        } else {
            return totalCost;
        }
    }
}
```

### 1.6.3 方法调用内存分析
- 方法没有被调用的时候，都在方法区中的字节码文件(.class)中存储。
- 方法被调用的时候，需要进入到栈内存中运行。方法每调用一次就会在栈中有一个入栈操作
- 当方法执行结束后，会释放该内存，称为出栈

### 1.6.4 方法的重载

* 方法签名 : 方法名+依次参数类型。注意，返回值不属于方法签名。方法签名是一个方法在一个类中的唯一标识
* 同一个类中方法可以重名，但是签名不可以重复。一个类中如果定义了名字相同，签名不同的方法，就叫做方法的重载

1. 重载的参数匹配规则

如果有如下重载方法，在java中的自动类型转换匹配逻辑为：
```java
public double buy(int count) {}
public double buy(double count){}
```
如果传的参数是 byte, short, int,类型会调用buy(int count)方法，如果是long, float, double 类型会调用buy(double count)方法。

- 无论是否重载参数类型可以不完全匹配的规则是"实参数可以自动类型转换成形参类型"
- 重载的特殊之处是，参数满足自动类型转换的方法有好几个，重载的规则是选择最"近"的去调用

### 1.6.5 可变个数的形参

格式：
```java
//JDK5.0:采用可变个数形参来定义方法，传入多个同一类型变量 
public static void test(int a ,String...books);
```
特点：
1. 方法的参数部分有可变形参，需要放在形参声明的最后
2. 在一个方法的形参中，最多只能声明一个可变个数的形参
3. 可变参数方法的使用与方法参数部分使用数组是一致的，二者不能同时声明，否则报错

代码示例：
```java
public class StringTools {
    String concat(char seperator, String... args){
        String str = "";
        for (int i = 0; i < args.length; i++) {
            if(i==0){
                str += args[i];
            }else{
                str += seperator + args[i];
			} 
		}
		return str; 
	}
}
```

### 1.6.6 参数传递机制：值传递
Java里方法的参数传递方式只有一种：值传递。
- 形参是基本数据类型：将实参基本数据类型变量的“数据值”传递给形参
- 形参是引用数据类型：将实参应用数据类型变量的“地址值”传递给形参
# 2 包和访问修饰符
## 2.1 package（包）

* package语句作为Java源文件的第一条语句出现。若缺省该语句，则指定为无名包。
* 不同的包里可以有相同名字的类
* 一个类只能有一个 package 语句，如果有 package 语句，则必须是类的第一行有效代码
* 包通常使用所在公司域名的倒置：`com.atguigu.xxx`。

```java
package 顶层包名.子包名 ;
```

**JDK中主要的包介绍：**
1. `java.lang` 包含一些Java语言的核心类，如 String、Math、Integer、System和Thread，提供常用功能
2. `java.net` 包含执行与网络相关的操作的类和接口
3. `java.io` 包含能提供多种输入、输出功能的类
4. `java.util` 包含一些实用工具类，如定义系统特性、接口的集合框架类、使用与日期日历相关的函数
5. `java.text` 包含一些java格式化相关的类
6. `java.sql` 包含了java 进行JDBC数据库编程的相关类、接口
7. `java.awt` 包含了构成抽象窗口工具类的多个类，这些类被用来构建和管理应用程序的图形用户界面（GUI）

## 2.2 import（导入）
1. 使用import

每次使用都带包名很繁琐， 可以在使用的类的上面使用 import 语句， 一次性解决问题， 就可以直接使用类了。

```java
import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;
```
- 如果使用 `a.*` 导入结构，表示可以导入a包下的所有的结构。举例:可以使用`java.util.*`的方式，一 次性导入util包下所有的类或接口。
- 如果导入的类或接口是`java.lang`包下的，或者是当前包下的，则可以省略此import语句。
- 如果已经导入java.a包下的类，那么如果需要使用a包的子包下的类的话，仍然需要导入。
- 如果在代码中使用不同包下的同名的类，那么就需要使用类的全类名的方式指明调用的是哪个类。
- `import static` 组合的使用:调用指定类或接口下的静态的属性或方法
## 2.2 访问修饰符

1. 属性访问修饰符：public

* 被 public 修饰的属性，可以被任意包中的类访问
* 没有访问修饰符的属性，称作缺省的访问修饰符，可以被本包内的其他类和自己的对象
* 访问修饰符是一种限制或者允许属性访问的修饰符

2. 属性访问修饰符：protected
- 被protected修饰，可以在本类使用
- 可以在本包子类和非子类使用
- 在其他包中仅限子类可见

3. 属性访问修饰符：priviate，仅可在本类可见
4. 属性访问修饰符：缺省，可在本类和本包子类非子类可见，其他包不可见

实现封装就是控制类或成员的可见性范围。这就需要依赖访问控制修饰符，也称为权限修饰符来控制。

| 修饰符       | 本类  | 本包            | 其他包子类          | 其他包非子类 |
| --------- | --- | ------------- | -------------- | ------ |
| private   | √   | ×             | ×              | ×      |
| 缺省        | √   | √（本包子类非子类都可见） | ×              | ×      |
| protected | √   | √（本包子类非子类都可见） | √（其他包仅限于子类中可见） | ×      |
| public    | √   | √             | √              | √      |
6. 类的全限定名

* 包名 + 类名 = 类的全限定名。也可以简称为类的全名
* 同一个 Java 程序中全限定名字不可重复
# 3 关键字：this

## 3.1 实例方法或构造器中使用当前对象的成员

当形参与成员变量同名时，如果在方法内或构造器内需要使用成员变量，必须添加this来表明该变量是类的成员变量。即：我们可以用this来区分`成员变量`和`局部变量`。比如：

![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240702144938.png)
> 使用this访问属性和方法时，如果在本类中未找到，会从父类中查找。

## 3.2 同一个类中构造器互相调用
- this()：调用本类的无参构造器
- this(实参列表)：调用本类的有参构造器
- this()和this(实参列表)只能声明在构造器首行。

```java
public class Student {
    private String name;
    private int age;

    // 无参构造
    public Student() {
//        this("",18);//调用本类有参构造器
    }

    // 有参构造
    public Student(String name) {
        this();//调用本类无参构造器
        this.name = name;
    }
    // 有参构造
    public Student(String name,int age){
        this(name);//调用本类中有一个String参数的构造器
        this.age = age;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    public String getInfo(){
        return "姓名：" + name +"，年龄：" + age;
    }
}

```

# 4 继承
## 4.1 继承的语法
通过 `extends` 关键字，可以声明一个类B继承另外一个类A，定义格式如下：
```java
[修饰符] class 类A {
	...
}

[修饰符] class 类B extends 类A {
	...
}
```

## 4.2 继承中的基本概念
类B，称为子类、派生类(derived class)、SubClass
类A，称为父类、超类、基类(base class)、SuperClass

## 4.3 继承性的细节
- 子类继承了父类的方法和属性
- 使用子类的引用可以调用父类的公有方法
- 使用子类的引用可以访问父类的公有属性
- 子类不能直接访问父类中私有的(private)的成员变量和方法，可通过继承的get/set方法进行访问。如图所示：

![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240702150117.png)

- Java只支持单继承，不支持多重继承

![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240702150258.png)

## 4.4 方法的重写（override/overwrite）

子类可以对从父类中继承来的方法进行改造，我们称为方法的`重写 (override、overwrite)`。也称为方法的`重置`、`覆盖`。
```java
package com.atguigu.inherited.method;

public class Phone {
    public void sendMessage(){
        System.out.println("发短信");
    }
    public void call(){
        System.out.println("打电话");
    }
    public void showNum(){
        System.out.println("来电显示号码");
    }
}
```

```java
package com.atguigu.inherited.method;

//SmartPhone：智能手机
public class SmartPhone extends Phone{
    //重写父类的来电显示功能的方法
	@Override
    public void showNum(){
        //来电显示姓名和图片功能
        System.out.println("显示来电姓名");
        System.out.println("显示头像");
    }
    //重写父类的通话功能的方法
    @Override
    public void call() {
        System.out.println("语音通话 或 视频通话");
    }
}
```

>@Override使用说明：
写在方法上面，用来检测是不是满足重写方法的要求。这个注解就算不写，只要满足要求，也是正确的方法覆盖重写。建议保留，这样编译器可以帮助我们检查格式，另外也可以让阅读源代码的程序员清晰的知道这是一个重写的方法。

1. 子类重写的方法`必须`和父类被重写的方法具有相同的`方法名称`、`参数列表`。
2. 子类重写的方法的返回值类型`不能大于`父类被重写的方法的返回值类型。（例如：Student < Person）。
> 注意：如果返回值类型是基本数据类型和void，那么必须是相同
3. 子类重写的方法使用的访问权限`不能小于`父类被重写的方法的访问权限。（public > protected > 缺省 > private）
> 注意：① 父类私有方法不能重写 ② 跨包的父类缺省的方法也不能重写
4. 子类方法抛出的异常不能大于父类被重写方法的异常
此外，子类与父类中同名同参数的方法必须同时声明为非static的(即为重写)，或者同时声明为static的（不是重写）。因为static方法是属于类的，子类无法覆盖父类的方法。
## 4.5 关键字：super
在Java类中使用super来调用父类中的指定操作：

- super可用于访问父类中定义的属性
- super可用于调用父类中定义的成员方法
- super可用于在子类构造器中调用父类的构造器

注意：

- 尤其当子父类出现同名成员时，可以用super表明调用的是父类中的成员
- super的追溯不仅限于直接父类
- super和this的用法相像，this代表本类对象的引用，super代表父类的内存空间的标识

### 4.5.1 子类中调用父类被重写的方法
- 如果子类没有重写父类的方法，只要权限修饰符允许，在子类中完全可以直接调用父类的方法；
- 如果子类重写了父类的方法，在子类中需要通过`super.`才能调用父类被重写的方法，否则默认调用的子类重写的方法

```java
package com.atguigu.inherited.method;

public class Phone {
    public void sendMessage(){
        System.out.println("发短信");
    }
    public void call(){
        System.out.println("打电话");
    }
    public void showNum(){
        System.out.println("来电显示号码");
    }
}

//smartphone：智能手机
public class SmartPhone extends Phone{
    //重写父类的来电显示功能的方法
    public void showNum(){
        //来电显示姓名和图片功能
        System.out.println("显示来电姓名");
        System.out.println("显示头像");

        //保留父类来电显示号码的功能
        super.showNum();//此处必须加super.，否则就是无限递归，那么就会栈内存溢出
    }
}
```
总结：
- **方法前面没有super.和this.**
	- 先从子类找匹配方法，如果没有，再从直接父类找，再没有，继续往上追溯
- **方法前面有this.**
    - 先从子类找匹配方法，如果没有，再从直接父类找，再没有，继续往上追溯
- **方法前面有super.**
    - 从当前子类的直接父类找，如果没有，继续往上追溯

### 4.5.2 子类中调用父类中同名的成员变量
- 如果实例变量与局部变量重名，可以在实例变量前面加this.进行区别
- 如果子类实例变量和父类实例变量重名，并且父类的该实例变量在子类仍然可见，在子类中要访问父类声明的实例变量需要在父类实例变量前加super.，否则默认访问的是子类自己声明的实例变量
- 如果父子类实例变量没有重名，只要权限修饰符允许，在子类中完全可以直接访问父类中声明的实例变量，也可以用this.实例访问，也可以用super.实例变量访问

### 4.5.3 子类构造器中调用父类构造器
① 子类继承父类时，不会继承父类的构造器。只能通过“super(形参列表)”的方式调用父类指定的构造器。
② 规定：“super(形参列表)”，必须声明在构造器的首行。
③ 我们前面讲过，在构造器的首行可以使用"this(形参列表)"，调用本类中重载的构造器，  
结合②，结论：在构造器的首行，"this(形参列表)" 和 "super(形参列表)"只能二选一。
④ 如果在子类构造器的首行既没有显示调用"this(形参列表)"，也没有显式调用"super(形参列表)"，  
​ 则子类此构造器默认调用"super()"，即调用父类中空参的构造器。
⑤ 由③和④得到结论：子类的任何一个构造器中，要么会调用本类中重载的构造器，要么会调用父类的构造器。  
只能是这两种情况之一。
⑥ 由⑤得到：一个类中声明有n个构造器，最多有n-1个构造器中使用了"this(形参列表)"，则剩下的那个一定使用"super(形参列表)"。

> 开发中常见错误：
> 如果子类构造器中既未显式调用父类或本类的构造器，且父类中又没有空参的构造器，则`编译出错`。

```java
class A{
	A(int a){
		System.out.println("A类有参构造器");
	}
}
class B extends A{
	B(){
		System.out.println("B类无参构造器");
	}
}
class Test05{
    public static void main(String[] args){
        B b = new B();
        //A类显示声明一个有参构造，没有写无参构造，那么A类就没有无参构造了
		//B类显示声明一个无参构造，        
		//B类的无参构造没有写super(...)，表示默认调用A类的无参构造
        //编译报错，因为A类没有无参构造
    }
}
```
![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240702152842.png)

## 4. 6 继承里的静态方法：

静态方法可以被继承，静态方法不支持多态逻辑，建议在使用静态方法时直接使用类名调用静态方法，如果此类下没有这个方法，则会调用其父类下的静态方法。

```java
package com.geekbang.supermarket;

public class Phone extends MerchandiseV2 {

    // 给Phone增加新的属性和方法
    private double screenSize;
    private double cpuHZ;
    private int memoryG;
    private int storageG;
    private String brand;
    private String os;
    private static int MAX_BUY_ONE_ORDER = 5;

    // >> TODO 使用super可以调用父类的方法和属性（当然必须满足访问控制符的控制）
    public double buy(int count) {
        if (count > MAX_BUY_ONE_ORDER) {
            System.out.println("购买失败，手机一次最多只能买" + MAX_BUY_ONE_ORDER + "个");
            return -2;
        }
        return super.buy(count);
    }

    public String getName() {
        return this.brand + ":" + this.os + ":" + name;
    }

    public void describe() {
        System.out.println("此手机商品属性如下");
        super.describe();
        System.out.println("手机厂商为" + brand + "；系统为" + os + "；硬件配置如下：\n" +
            "屏幕：" + screenSize + "寸\n" +
            "cpu主频" + cpuHZ + " GHz\n" +
            "内存" + memoryG + "Gb\n" +
            "存储空间" + storageG + "Gb");
    }

    // >> TODO super是子类和父类交流的桥梁，但是并不是父类的引用
    // >> TODO 所以，super和this自引用不一样，不是简单可以模拟的（可以模拟的话不就成了组合了吗）
//    public MerchandiseV2 getParent(){
//        return super;
//    }

    public Phone getThisPhone(){
        return this;
    }

    // >> TODO 使用super可以调用父类的public属性，但是super不是一个引用。
    public void accessParentProps() {
        System.out.println("父类里的name属性：" + super.name);
    }

    public void useSuper() {
        // >> TODO super的用法就像是一个父类的引用。它是继承的一部分，像组合的那部分，但不是全部
        super.describe();
        super.buy(66);
        System.out.println("父类里的count属性：" + super.count);
    }

    public Phone(
        String name, String id, int count, double soldPrice, double purchasePrice,
        double screenSize, double cpuHZ, int memoryG, int storageG, String brand, String os
    ) {
        // >> TODO 可以认为，创建子类对象的时候，也就同时创建了一个隐藏的父类对象

        this.screenSize = screenSize;
        this.cpuHZ = cpuHZ;
        this.memoryG = memoryG;
        this.storageG = storageG;
        this.brand = brand;
        this.os = os;

        // >> TODO 所以，才能够setName，对name属性进行操作。
        this.setName(name);
        this.setId(id);
        this.setCount(count);
        this.setSoldPrice(soldPrice);
        this.setPurchasePrice(purchasePrice);
    }

    public boolean meetCondition() {
        return true;
    }

    public double getScreenSize() {
        return screenSize;
    }

    public void setScreenSize(double screenSize) {
        this.screenSize = screenSize;
    }

    public double getCpuHZ() {
        return cpuHZ;
    }

    public void setCpuHZ(double cpuHZ) {
        this.cpuHZ = cpuHZ;
    }

    public int getMemoryG() {
        return memoryG;
    }

    public void setMemoryG(int memoryG) {
        this.memoryG = memoryG;
    }

    public int getStorageG() {
        return storageG;
    }

    public void setStorageG(int storageG) {
        this.storageG = storageG;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getOs() {
        return os;
    }

    public void setOs(String os) {
        this.os = os;
    }
}

```

## 4.7 父类和子类的引用赋值关系
- 父类引用可以指向子类对象，子类引用不可以指向父类的对象
- 可以进行强制类型转换，如果类型不对，会报错
- 可以调用的方法，是受引用类型决定的

* 因为子类继承了父类的方法和属性，所以父类的对象能做到的，子类的对象肯定能做到
* 如果确定一个父类的引用指向的对象，实际上就是一个子类的对象（或者子类的子类的对象），可以强制类型转换，如果不是，则编译报错

**如下示例：**
类之间的关系为：LittleSuperMarket -> MerchandiseV2 -> Phone -> ShellColorChangePhone

```java
package com.geekbang;

import com.geekbang.supermarket.MerchandiseV2;
import com.geekbang.supermarket.Phone;
import com.geekbang.supermarket.ShellColorChangePhone;

public class ReferenceAssign {
    public static void main(String[] args) {
        Phone ph = new Phone(
            "手机001", "Phone001", 100, 1999, 999,
            4.5, 3.5, 4, 128, "索尼", "安卓"
        );

        // >> TODO 可以用子类的引用给父类的引用赋值，也就是说，父类的引用可以指向子类的对象

        MerchandiseV2 m = ph;
        MerchandiseV2 m2 = new Phone(
            "手机002", "Phone002", 100, 1999, 999,
            4.5, 3.5, 4, 128, "索尼", "安卓"
        );

        // >> TODO 但是反之则不行，不能让子类的引用指向父类的对象。因为父类并没有子类的属性和方法呀

//         Phone notDoable = new MerchandiseV2();

        // >> TODO                          重点
        // >> TODO 因为子类继承了父类的方法和属性，所以父类的对象能做到的，子类的对象肯定能做到
        //    TODO 换句话说，我们可以在子类的对象上，执行父类的方法
        // >> TODO 当父类的引用指向子类的实例（或者父类的实例），只能通过父类的引用，像父类一样操作子类的对象
        //    TODO 也就是说"名"的类型，决定了能执行哪些操作


        // >> TODO ph和m都指向同一个对象，通过ph可以调用getBrand方法
        //    TODO 因为ph的类型是Phone，Phone里定义了getBrand方法
        ph.getBrand();
        // >> TODO ph和m都指向同一个对象，但是通过m就不可以调用getBrand方法
        //    TODO 因为m的类型是MerchandiseV2，MerchandiseV2里没有你定义getBrand方法
        // m.getBrand();

        // TODO 如果确定一个父类的引用指向的对象，实际上就是一个子类的对象（或者子类的子类的对象），可以强制类型转换
        Phone aPhone = (Phone) m2;

        // MerchandiseV2是Phone的父类，Phone是shellColorChangePhone的父类
        ShellColorChangePhone shellColorChangePhone = new ShellColorChangePhone(
            "手机002", "Phone002", 100, 1999, 999,
            4.5, 3.5, 4, 128, "索尼", "安卓"
        );

        // TODO 父类的引用，可以指向子类的对象，即可以用子类（以及子类的子类）的引用给父类的引用赋值
        MerchandiseV2 ccm = shellColorChangePhone;

        // TODO 父类的引用，可以指向子类的对象。
        // TODO 确定MerchandiseV2的引用ccm是指向的是Phone或者Phone的子类对象，那么可以强制类型转换
        Phone ccp = (Phone) ccm;

        // TODO 确定MerchandiseV2的引用ccm是指向的是ShellColorChangePhone或者ShellColorChangePhone的子类对象
        // TODO 那么可以强制类型转换
        ShellColorChangePhone scp = (ShellColorChangePhone) ccm;

        // TODO 会出错，因为m2指向的是一个Phone类型的对象，不是ShellColorChangePhone的对象
        ShellColorChangePhone notCCP = (ShellColorChangePhone) m2;


    }
}

```


# 5 多态性
## 5.1 多态的形式和体现
对象的多态：在Java中，子类的对象可以替代父类的对象使用。所以，一个引用类型变量可能指向(引用)多种不同类型的对象

Java引用变量有两个类型：`编译时类型`和`运行时类型`。编译时类型由`声明`该变量时使用的类型决定，运行时类型由`实际赋给该变量的对象`决定。简称：**编译时，看左边；运行时，看右边。**

- 若编译时类型和运行时类型不一致，就出现了对象的多态性(Polymorphism)
- 多态情况下，“看左边”：看的是父类的引用（父类中不具备子类特有的方法）  
    “看右边”：看的是子类的对象（实际运行的是子类重写父类的方法）

多态的使用前提：1. 类的继承关系 2. 方法的重写

**举例：**

```java
package com.atguigu.polymorphism.grammar;

public class Pet {
    private String nickname; //昵称

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public void eat(){
        System.out.println(nickname + "吃东西");
    }
}
```

```java
package com.atguigu.polymorphism.grammar;

public class Cat extends Pet {
    //子类重写父类的方法
    @Override
    public void eat() {
        System.out.println("猫咪" + getNickname() + "吃鱼仔");
    }

    //子类扩展的方法
    public void catchMouse() {
        System.out.println("抓老鼠");
    }
}
```

```java
package com.atguigu.polymorphism.grammar;

public class Dog extends Pet {
    //子类重写父类的方法
    @Override
    public void eat() {
        System.out.println("狗子" + getNickname() + "啃骨头");
    }

    //子类扩展的方法
    public void watchHouse() {
        System.out.println("看家");
    }
}
```

**1、方法内局部变量的赋值体现多态**

```java
package com.atguigu.polymorphism.grammar;

public class TestPet {
    public static void main(String[] args) {
        //多态引用
        Pet pet = new Dog();
        pet.setNickname("小白");

        //多态的表现形式
        /*
        编译时看父类：只能调用父类声明的方法，不能调用子类扩展的方法；
        运行时，看“子类”，如果子类重写了方法，一定是执行子类重写的方法体；
         */
        pet.eat();//运行时执行子类Dog重写的方法
//      pet.watchHouse();//不能调用Dog子类扩展的方法

        pet = new Cat();
        pet.setNickname("雪球");
        pet.eat();//运行时执行子类Cat重写的方法
    }
}
```

**2、方法的形参声明体现多态**

```java
package com.atguigu.polymorphism.grammar;

public class Person{
    private Pet pet;
    public void adopt(Pet pet) {//形参是父类类型，实参是子类对象
        this.pet = pet;
    }
    public void feed(){
        pet.eat();//pet实际引用的对象类型不同，执行的eat方法也不同
    }
}
```

```java
package com.atguigu.polymorphism.grammar;

public class TestPerson {
    public static void main(String[] args) {
        Person person = new Person();

        Dog dog = new Dog();
        dog.setNickname("小白");
        person.adopt(dog);//实参是dog子类对象，形参是父类Pet类型
        person.feed();

        Cat cat = new Cat();
        cat.setNickname("雪球");
        person.adopt(cat);//实参是cat子类对象，形参是父类Pet类型
        person.feed();
    }
}
```

**3、方法返回值类型体现多态**

```java
package com.atguigu.polymorphism.grammar;

public class PetShop {
    //返回值类型是父类类型，实际返回的是子类对象
    public Pet sale(String type){
        switch (type){
            case "Dog":
                return new Dog();
            case "Cat":
                return new Cat();
        }
        return null;
    }
}
```

```java
package com.atguigu.polymorphism.grammar;

public class TestPetShop {
    public static void main(String[] args) {
        PetShop shop = new PetShop();

        Pet dog = shop.sale("Dog");
        dog.setNickname("小白");
        dog.eat();

        Pet cat = shop.sale("Cat");
        cat.setNickname("雪球");
        cat.eat();
    }
}
```

## 5.2 多态的好处和弊端
**好处**：变量引用的子类对象不同，执行的方法就不同，实现动态绑定。代码编写更灵活、功能更强大，可维护性和扩展性更好了。
**弊端**：一个引用类型变量如果声明为父类的类型，但实际引用的是子类对象，那么该变量就不能再访问子类中添加的属性和方法。
```java
Student m = new Student();  
m.school = "pku"; //合法,Student类有school成员变量  
Person e = new Student();   
e.school = "pku"; //非法,Person类没有school成员变量  
​  
// 属性是在编译时确定的，编译时e为Person类型，没有school成员变量，因而编译错误。
```

开发中：使用父类做方法的形参，是多态使用最多的场合。即使增加了新的子类，方法也无需改变，提高了扩展性，符合开闭原则。

## 5.3 虚方法调用
在Java中虚方法是指在编译阶段不能确定方法的调用入口地址，在运行阶段才能确定的方法，即可能被重写的方法。

```java
Person e = new Student();  
e.getInfo();  //调用Student类的getInfo()方法
```
子类中定义了与父类同名同参数的方法，在多态情况下，将此时父类的方法称为虚方法，父类根据赋给它的不同子类对象，动态调用属于子类的该方法。这样的方法调用在编译期是无法确定的。
![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240702154525.png)
前提：Person类中定义了welcome()方法，各个子类重写了welcome()。

拓展：
`静态链接（或早起绑定）`：当一个字节码文件被装载进JVM内部时，如果被调用的目标方法在编译期可知，且运行期保持不变时。这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接。那么调用这样的方法，就称为非虚方法调用。比如调用静态方法、私有方法、final方法、父类构造器、本类重载构造器等。
`动态链接（或晚期绑定）`：如果被调用的方法在编译期无法被确定下来，也就是说，只能够在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接。调用这样的方法，就称为虚方法调用。比如调用重写的方法（针对父类）、实现的方法（针对接口）。

## 5.5 instanceof操作符
instanceof 操作符，可以判断一个引用指向的对象是否是某一个类型或者其子类，是则返回true，否则返回false。

```java
package com.geekbang;

import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;
import com.geekbang.supermarket.Phone;
import com.geekbang.supermarket.ShellColorChangePhone;

public class InstanceOfTestAppMain {
    public static void main(String[] args) {
        int merchandiseCount = 600;
        LittleSuperMarket superMarket = new LittleSuperMarket("大卖场",
            "世纪大道1号", 500, merchandiseCount, 100);

        // >> TODO instanceof 操作符，可以判断一个引用指向的对象是否是某一个类型或者其子类
        //    TODO 是则返回true，否则返回false
        for(int i =0;i<merchandiseCount;i++){
            MerchandiseV2 m = null;// superMarket.getMerchandiseOf(i);
            if(m instanceof MerchandiseV2){
                // TODO 先判断，再强制类型转换，比较安全
                MerchandiseV2 ph = (MerchandiseV2)m;
                System.out.println(ph.getName());
            }else {
                System.out.println("not an instance");
            }
        }

        // >> TODO 如果引用是null，则肯定返回false


    }
}
```

# 6 Object类的使用
## 6.1 根父类
所有的类，都直接或间接地继承自Object类。
Object类的对象可以指向任意类，Object类中没有属性，只有方法，定义的对象如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202308081038318.png)

## 6.2 Object类中的方法：
### 6.2.1 equals

在Object类中定义判断两个对象的逻辑如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202308081043733.png)
比较两个引用是否指向的是相同的类实例。

equals方法也是所有自定义类中覆盖实现比较多的方法，通过重新实现equals方法的逻辑，比较自定义类的对象。

- **String类中对Object的使用**

**程序示例：**
```java
package com.geekbang;

import com.geekbang.supermarket.LittleSuperMarket;

import java.util.Scanner;

public class StringEqualsAppMain {
    public static void main(String[] args) {

        LittleSuperMarket superMarket = new LittleSuperMarket("大卖场",
            "世纪大道1号", 500, 600, 100);

        String s1 = "aaabbb";

        String s2 = "aaa" + "bbb";

        // >> TODO 说好的每次创建一个新的String对象呢？
        System.out.println("s1和s2用==判断结果："+(s1 == s2));

        System.out.println("s1和s2用 equals 判断结果："+(s1.equals(s2)));

        // >> TODO 打乱Java对String的的优化，再试试看
        Scanner scanner = new Scanner(System.in);

        System.out.println("请输入s1");
        s1 = scanner.nextLine();

        System.out.println("请输入s2");
        s2 = scanner.nextLine();

        System.out.println("s1和s2用==判断结果："+(s1 == s2));

        System.out.println("s1和s2用 equals 判断结果："+(s1.equals(s2)));
    }

}

```
输出：
```
s1和s2用==判断结果：true
s1和s2用 equals 判断结果：true
请输入s1
C:\Users\baihaoliang\.jdks\corretto-1.8.0_382\bin\java.exe -javaagent:D:\JetBrains\apps\IDEA-U\ch-0\231.9225.16\lib\idea_rt.jar=12228:D:\JetBrains\apps\IDEA-U\ch-0\231.9225.16\bin -Dfile.encoding=U
请输入s2
C:\Users\baihaoliang\.jdks\corretto-1.8.0_382\bin\java.exe -javaagent:D:\JetBrains\apps\IDEA-U\ch-0\231.9225.16\lib\idea_rt.jar=12228:D:\JetBrains\apps\IDEA-U\ch-0\231.9225.16\bin -Dfile.encoding=U
s1和s2用==判断结果：false
s1和s2用 equals 判断结果：true
```

String中定义的字符串是不可变的，所以按说在定义s1和s2时应该是不同的两个引用，但是在使用"\=="进行比较时发现是相等的。这是因为在Java中定义字符串时，如果底层有一个相同的字符串，则不会重新定义，会把新的引用指向之前的字符串。
但是在定义的字符串很长时，则会突破这个优化，如上，输入了一个很长的字符串，使用`"=="`比较是不相同的，但是使用equals是相同的，下边看下String类中对equals方法的覆盖实现：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202308081053010.png)
String中重新实现了equals方法按字符进行遍历比较，所以再长的字符串的情况下依然可以比较。

**面试题：**`==`和equals的区别
- `==` 既可以比较基本类型也可以比较引用类型。对于基本类型就是比较值，对于引用类型就是比较内存地址
- equals的话，它是属于java.lang.Object类里面的方法，如果该方法没有被重写过默认也是`==`;我们可以看到String等类的equals方法是被重写过的，而且String类在日常开发中用的比较多，久而久之，形成了equals是比较值的错误观点。
- 具体要看自定义类里有没有重写Object的equals方法来判断。
- 通常情况下，重写equals方法，会比较类中的相应属性是否都相等。
### 6.2.2 hashCode

hashCode可以翻译为哈希码，或者散列码。应该是一个表示对象的特征的int整数。
自定义类中可以进行覆盖实现，来表示依次来表示类的唯一标识。

> equals和hashCode是最常覆盖的两个方法，实现逻辑为，equals方法为true，则hashCode就相等，但hashCode相等，equals不一定为true。

### 6.2.3 toString
方法签名：public String toString()
① 默认情况下，toString()返回的是“对象的运行时类型 @ 对象的hashCode值的十六进制形式"
② 在进行String与其它类型数据的连接操作时，自动调用toString()方法

```java
Date now=new Date();
System.out.println(“now=”+now);  //相当于
System.out.println(“now=”+now.toString()); 
```

③ 如果我们直接System.out.println(对象)，默认会自动调用这个对象的toString()

> 因为Java的引用数据类型的变量中存储的实际上时对象的内存地址，但是Java对程序员隐藏内存地址信息，所以不能直接将内存地址显示出来，所以当你打印对象时，JVM帮你调用了对象的toString()。

④ 可以根据需要在用户自定义类型中重写toString()方法
	如String 类重写了toString()方法，返回字符串的值。

```java
s1="hello";
System.out.println(s1);//相当于System.out.println(s1.toString());
```

例如自定义的Person类：
```java
public class Person {  
    private String name;
    private int age;

    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

### 6.2.4 finalize()
- 当对象被回收时，系统自动调用该对象的 finalize() 方法。（不是垃圾回收器调用的，是本类对象调用的）
  - 永远不要主动调用某个对象的finalize方法，应该交给垃圾回收机制调用。
- 什么时候被回收：当某个对象没有任何引用时，JVM就认为这个对象是垃圾对象，就会在之后不确定的时间使用垃圾回收机制来销毁该对象，在销毁该对象前，会先调用 finalize()方法。 
- 子类可以重写该方法，目的是在对象被清理之前执行必要的清理操作。比如，在方法内断开相关连接资源。
  - 如果重写该方法，让一个新的引用变量重新引用该对象，则会重新激活对象。
- 在JDK 9中此方法已经被`标记为过时`的。

```java
public class FinalizeTest {
	public static void main(String[] args) {
		Person p = new Person("Peter", 12);
		System.out.println(p);
		p = null;//此时对象实体就是垃圾对象，等待被回收。但时间不确定。
		System.gc();//强制性释放空间
	}
}

class Person{
	private String name;
	private int age;

	public Person(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	//子类重写此方法，可在释放对象前进行某些操作
	@Override
	protected void finalize() throws Throwable {
		System.out.println("对象被释放--->" + this);
	}
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]";
	}
	
}
```

### 6.2.5 getClass()

`public final Class<?> getClass()`：获取对象的运行时类型

> 因为Java有多态现象，所以一个引用数据类型的变量的编译时类型与运行时类型可能不一致，因此如果需要查看这个变量实际指向的对象的类型，需要用getClass()方法

```java
public static void main(String[] args) {
	Object obj = new Person();
	System.out.println(obj.getClass());//运行时类型
}
```
结果：
```java
class com.atguigu.java.Person
```

# 7 关键字：static
## 7.1 static 关键字
- 使用范围：
    - 在Java类中，可用static修饰属性、方法、代码块、内部类
- 被修饰后的成员具备以下特点：
    - 随着类的加载而加载
    - 优先于对象存在
    - 修饰的成员，被所有对象所共享
    - 访问权限允许时，可不创建对象，直接被类调用

## 7.2 静态变量
- 静态变量的默认值规则和实例变量一样。
- 静态变量值是所有对象共享。
- 静态变量在本类中，可以在任意方法、代码块、构造器中直接使用。
- 如果权限修饰符允许，在其他类中可以通过“`类名.静态变量`”直接访问，也可以通过“`对象.静态变量`”的方式访问（但是更推荐使用类名.静态变量的方式）。
- 静态变量的get/set方法也静态的，当局部变量与静态变量`重名时`，使用“`类名.静态变量`”进行区分。

## 7.3 静态方法
- 静态方法在本类的任意方法、代码块、构造器中都可以直接被调用。
- 只要权限修饰符允许，静态方法在其他类中可以通过“类名.静态方法“的方式调用。也可以通过”对象.静态方法“的方式调用（但是更推荐使用类名.静态方法的方式）。
- 在static方法内部只能访问类的static修饰的属性或方法，不能访问类的非static的结构。
- 静态方法可以被子类继承，但不能被子类重写。
- 静态方法的调用都只看编译时类型。
- 因为不需要实例就可以访问static方法，因此static方法内部不能有this，也不能有super。如果有重名问题，使用“类名.”进行区别。

**示例：**
```java
package com.atguigu.keyword;

public class Father {
    public static void method(){
        System.out.println("Father.method");
    }

    public static void fun(){
        System.out.println("Father.fun");
    }
}
```

```java
package com.atguigu.keyword;

public class Son extends Father{
//    @Override //尝试重写静态方法，加上@Override编译报错，去掉Override不报错，但是也不是重写
    public static void fun(){
        System.out.println("Son.fun");
    }
}
```

```java
package com.atguigu.keyword;

public class TestStaticMethod {
    public static void main(String[] args) {
        Father.method();
        Son.method();//继承静态方法

        Father f = new Son();
        f.method();//执行Father类中的method
    }
}
```

# 8 代码块
## 8.1 静态代码块
1. 可以有输出语句。
2. 可以对类的属性、类的声明进行初始化操作。
3. 不可以对非静态的属性初始化。即：**不可以调用非静态的属性和方法**。
4. 若有多个静态的代码块，那么按照从上到下的顺序依次执行。
5. 静态代码块的执行要先于非静态代码块。
6. 静态代码块随着类的加载而加载，且只执行一次。

```java
package com.atguigu.keyword;

public class Chinese {
//    private static String country = "中国";

    private static String country;
    private String name;

    {
        System.out.println("非静态代码块，country = " + country);
    }

    static {
        country = "中国";
        System.out.println("静态代码块");
    }

    public Chinese(String name) {
        this.name = name;
    }
}
```
## 8.2 非静态代码块
如果多个重载的构造器有公共代码，并且这些代码都是先于构造器其他代码执行的，那么可以将这部分代码抽取到非静态代码块中，减少冗余代码。

1. 可以有输出语句。
2. 可以对类的属性、类的声明进行初始化操作。
3. 除了调用非静态的结构外，还可以调用静态的变量或方法。
4. 若有多个非静态的代码块，那么按照从上到下的顺序依次执行。
5. 每次创建对象的时候，都会执行一次。且先于构造器执行。

# 9 final 修饰符
## 9.1 final修饰类：不可被继承
```java
final class Eunuch{//太监类
	
}
class Son extends Eunuch{//错误
	
}
```

## 9.2 final修饰方法：不可被子类覆盖
```java
class Father{
	public final void method(){
		System.out.println("father");
	}
}
class Son extends Father{
	public void method(){//错误
		System.out.println("son");
	}
}
```

## 9.3 final 修饰变量
final修饰某个变量（成员变量或局部变量），一旦赋值，它的值就不能被修改，即常量，常量名建议使用大写字母。
例如：`final double MY_PI = 3.14;`

1. 构造方法不能用final修饰
2. 使用final修饰方法的形参后，则无法在方法内修改形参变量

1. 修饰静态变量：需要初始化，或者在static块中初始化，并且初始化后无法修改。

```java
private final static int MAX_BUY_ONE_ORDER = 9;
//或   选其一
static {
    MAX_BUY_ONE_ORDER = 1;
}

```
2. 修饰属性：只能在定义时初始化或者构造方法中初始化，初始化后无法修改。
```java
public final class Test {
    public static int totalNumber = 5;
    public final int ID;

    public Test() {
        ID = ++totalNumber; // 可在构造器中给final修饰的“变量”赋值
    }
    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.ID);
    }
}
```

1. 修饰引用：引用类型本身初始化后无法修改，但可以通过引用修改引用的对象内容。
```java
int[] array1 = new int[10];
final int[] array = new int[10];
array = array1;  //报错，无法修改final变量
for(int b : array) {  //for循环的一种新形式
    System.out.println(b);
}
```
4. 修饰局部变量：只能初始化一次，后边不能再次修改
```java
public class TestFinal {
    public static void main(String[] args){
        final int MIN_SCORE ;
        MIN_SCORE = 0;
        final int MAX_SCORE = 100;
        MAX_SCORE = 200; //非法
    }
}
```

# 10 抽象类与抽象方法
## 10.1 语法格式
* **抽象类**：被abstract修饰的类。
* **抽象方法**：被abstract修饰没有方法体的方法。

抽象类的语法格式
```java
[权限修饰符] abstract class 类名{
    
}
[权限修饰符] abstract class 类名 extends 父类{
    
}
```

抽象方法的语法格式
```java
[其他修饰符] abstract 返回值类型 方法名([形参列表]);
```
> 注意：抽象方法没有方法体

代码举例：
```java
public abstract class Animal {
    public abstract void eat();
}
```

```java
public class Cat extends Animal {
    public void eat (){
      	System.out.println("小猫吃鱼和猫粮"); 
    }
}
```

```java
public class CatTest {
 	 public static void main(String[] args) {
        // 创建子类对象
        Cat c = new Cat(); 
       
        // 调用eat方法
        c.eat();
  	}
}
```

此时的方法重写，是子类对父类抽象方法的完成实现，我们将这种方法重写的操作，也叫做**实现方法**。

## 10.2 使用说明
1. 抽象类**不能创建对象**，如果创建，编译无法通过而报错。只能创建其非抽象子类的对象。
> 理解：假设创建了抽象类的对象，调用抽象的方法，而抽象方法没有具体的方法体，没有意义。
   抽象类是用来被继承的，抽象类的子类必须重写父类的抽象方法，并提供方法体。若没有重写全部的抽象方法，仍为抽象类。
2. 抽象类中，也有构造方法，是供子类创建对象时，初始化父类成员变量使用的。
>理解：子类的构造方法中，有默认的super()或手动的super(实参列表)，需要访问父类构造方法。
3. 抽象类中，不一定包含抽象方法，但是有抽象方法的类必定是抽象类。
> 理解：未包含抽象方法的抽象类，目的就是不想让调用者创建该类对象，通常用于某些特殊的类结构设计。
4. 抽象类的子类，必须重写抽象父类中**所有的**抽象方法，否则，编译无法通过而报错。除非该子类也是抽象类。 
> 理解：假设不重写所有抽象方法，则类中可能包含抽象方法。那么创建对象后，调用抽象的方法，没有意义。
## 10.3 注意事项
- 不能用abstract修饰变量、代码块、构造器；
- 不能用abstract修饰私有方法、静态方法、final的方法、final的类。

# 11 接口（interface）
##  接口（interface）
1. 接口的使用

* 接口的定义使用interface，而非class
* 接口中的方法，就是这个类型的规范，接口专注于规范，怎么实现这些规范，它不管
* 接口无法被实例化，也就是不可以new一个接口的实例。
* 接口里的方法都是public abstract修饰的，方法有名字，参数和返回值，没有方法体，以分号;结束。
* 因为接口里的方法都是且只能用public abstract修饰，所以这俩修饰符可以省略,abstract就是抽象方法的修饰符，没有方法体，以分号结束
* 接口里不能定义局部变量，定义的变量默认都是public static final的，这三个修饰符同样可以省略

**接口的定义：**
```java
package com.geekbang.supermarket;

import java.util.Date;

// >> TODO 接口的定义使用interface，而非class
// >> TODO 接口中的方法，就是这个类型的规范，接口专注于规范，怎么实现这些规范，它不管
// >> TODO 接口无法被实例话，也就是不可以new一个接口的实例。
public interface ExpireDateMerchandise {

    // >> TODO 接口里的方法都是public abstract修饰的，方法有名字，参数和返回值，没有方法体，以分号;结束，
    // TODO 接口注释最好写一下
    /**
     * 截止到当前，商品的保质期天数是否超过传递的天数
     *
     * @param days 截止到当前，保质期超过这么多天
     * @return 截止到当前，true如果保质期剩余天数比参数长，false如果保质期不到这多天
     */
    boolean notExpireInDays(int days);

    // >> TODO 因为接口里的方法都是且只能用public abstract修饰，所以这俩修饰符可以省略
    // >> TODO abstract就是抽象方法的修饰符，没有方法体，以分号结束
    /**
     * @return 商品生产日期
     */
    Date getProducedDate();

    /**
     * @return 商品保质期到期日
     */
    public abstract Date getExpireDate();

    /**
     * @return 截止到当前，剩余保质期还剩下总保质期长度的百分比
     */
    double leftDatePercentage();


    /**
     * 根据剩余的有效期百分比，得出商品现在实际的价值
     * @param leftDatePercentage 剩余有效期百分比
     * @return 剩余的实际价值
     */
    double actualValueNow(double leftDatePercentage);

    // >> TODO 接口里不能定义局部变量，定义的变量默认都是public static final的，这三个修饰符同样可以省略

    public static final int VAL_IN_INTERFACE = 999;

}

```
类可以继承接口，并且不像继承类一样只能继承一个父类，接口可以继承多个。
```java
public class GamePointCard extends MerchandiseV2 implements ExpireDateMerchandise, VirtualMerchandise {

}
```
以上GamePointCard类继承了MerchandiseV2，并且继承了ExpireDateMerchandise和VirtualMerchandise两个接口。
类中会实现继承的接口里边的方法。
因为类实现了接口的方法，可以用实现接口的类的引用，给接口的引用赋值。类似于可以使用子类的引用给父类赋值。
**接口的使用：**
```java
package com.geekbang;


import com.geekbang.supermarket.ExpireDateMerchandise;
import com.geekbang.supermarket.GamePointCard;
import com.geekbang.supermarket.MerchandiseV2;
import com.geekbang.supermarket.VirtualMerchandise;

import java.util.Date;

public class UseInterface {

    public static void main(String[] args) {

        Date produceDate = new Date();
        Date expireDate = new Date(produceDate.getTime() + 365L * 24 * 3600 * 1000);
        GamePointCard gamePointCard = new GamePointCard(
            "手机001", "Phone001", 100, 1999, 999,
            produceDate, expireDate
        );

        // phoneExtendsMerchandise.describe();


        // >> TODO 可以用实现接口的类的引用，给接口的引用赋值。类似于可以使用子类的引用给父类赋值
        ExpireDateMerchandise expireDateMerchandise = gamePointCard;

        VirtualMerchandise virtual = gamePointCard;

        MerchandiseV2 m = gamePointCard;

        expireDateMerchandise = (ExpireDateMerchandise) m;

        virtual = (VirtualMerchandise) m;

        if(m instanceof ExpireDateMerchandise){
            System.out.println("m 是 ExpireDateMerchandise 类型的实例");
        }

        if(m instanceof VirtualMerchandise){
            System.out.println("m 是 VirtualMerchandise 类型的实例");
        }


    }
}

```
以上定义的gamePointCard对象可以赋值给父类的引用，接口的引用。

2. 接口的继承

**Intf1接口：**
```
package com.geekbang.intf;

public interface Intf1 {
    void m1();
}
```

**Intf2接口：**
```java
package com.geekbang.intf;

public interface Intf2 {
    void m1();

    void m2();

}
```

**Intf3接口：**
```java
package com.geekbang.intf;

// >> TODO 接口也可以继承接口。接口可以继承多个接口，接口之间的继承要用extends
// >> TODO 接口不可以继承类
// >> TODO 继承的接口，可以有重复的方法，但是签名相同时，返回值必须完全一样，否则会有编译错误
public interface Intf3 extends Intf1, Intf2{
    void m3();
}
```

3. 接口的知识点

* 接口不可以继承类
* 接口不可以声明实例变量
* 如果一个类继承了两个接口，并且两个接口中的有相同的缺省方法，则编译报错。
* 接口中可以有静态方法，不需要用default修饰。静态方法可以被实现接口的类继承
* 接口中的this自引用，this引用的是实现这个接口的类所定义的实例



### 2.28 有方法的代码的接口

* 在Java 8中，接口允许有缺省实现的抽象方法。
* 缺省的实现方法，用default修饰，可以有方法体
* 接口中可以有私有方法，不需要用default修饰

```java
package com.geekbang.supermarket;

import java.util.Date;

public interface ExpireDateMerchandise {

    // >> TODO 缺省的实现方法，用default修饰，可以有方法体
    /**
     * 截止到当前，商品的保质期天数是否超过传递的天数
     *
     * @param days 截止到当前，保质期超过这么多天
     * @return 截止到当前，true如果保质期剩余天数比参数长，false如果保质期不到这多天
     */
    default boolean notExpireInDays(int days) {
        return daysBeforeExpire() > days;
    }

    /**
     * @return 商品生产日期
     */
    Date getProducedDate();

    /**
     * @return 商品保质期到期日
     */
    public abstract Date getExpireDate();

    /**
     * @return 截止到当前，剩余保质期还剩下总保质期长度的百分比
     */
    default double leftDatePercentage() {
        return 1.0 * daysBeforeExpire() / (daysBeforeExpire() + daysAfterProduce());
    }

    /**
     * 根据剩余的有效期百分比，得出商品现在实际的价值
     *
     * @param leftDatePercentage 剩余有效期百分比
     * @return 剩余的实际价值
     */
    double actualValueNow(double leftDatePercentage);

    // >> TODO 接口中可以有私有方法，不需要用default修饰
    // >> TODO 接口里的私有方法，可以认为是代码直接插入到使用的地方
    private long daysBeforeExpire() {
        long expireMS = getExpireDate().getTime();
        long left = expireMS - System.currentTimeMillis();
        if (left < 0) {
            return -1;
        }
        // 返回值是long，是根据left的类型决定的
        return left / (24 * 3600 * 1000);
    }

    private long daysAfterProduce() {
        long produceMS = getProducedDate().getTime();
        long left = System.currentTimeMillis() - produceMS;
        if (left < 0) {
            return -1;
        }
        // 返回值是long，是根据left的类型决定的
        return left / (24 * 3600 * 1000);
    }

}
```
面对有缺省方法的接口，一个类继承时可以有三种选择：
1. 默默继承，相当于类具有了这个方法的实现
2. 覆盖，重新实现
3. 把此方法声明为abstract，相当于把这个方法的实现拒之门外，但有abstrace方法的类，此类也就成了抽象类。


# 12 Math和Scanner的使用
- Scanner

```java
package com.geekbang.learn;

import java.math.BigDecimal;
import java.math.BigInteger;
import java.util.Scanner;

public class LearnScanner {
    public static void main(String[] args) {
        // TODO Scanner是一个方便的可以帮我们从标准输入读取并转换数据的类
        // TODO 注释里 @since   1.5 表示它是从Java5才开始有的。
        Scanner scanner = new Scanner(System.in);

        // TODO 但是这并是说从Java5开始，这个类就没有变化过了
        // TODO 在源代码里搜索一下@since，会发现很多方法是在后续的 Java 版本中加进去的
        // TODO 但是private方法就不会有这个文档标示，因为private方法本来就不给用。

        System.out.println("请输入一个巨大的正数");
        BigInteger bigInteger = scanner.nextBigInteger();
        System.out.println("请输入想给这个数加多少");
        BigInteger toBeAdd = scanner.nextBigInteger();
        System.out.println("结果为：" + bigInteger.add(toBeAdd));

    }
}

```

- Math

```java
package com.geekbang.learn;


import java.util.Random;

public class LearnMath {

    public double abc;

    public static void main(String[] args) {

        // TODO 我们调用的都是 Math 里的静态方法，Math的构造函数就是private的，意味着不能创建Math类的实例
        System.out.println(Math.random());
        // TODO 原来归根结底，Math的random是用的Random类来实现的。它在java.util包里
        Random random = new Random();
        for (int i = 0; i < 5; i++) {
            // TODO nextInt的返回值竟然有正数有负数哦！所以使用别人的类之前，一定要看看文档，避免出问题
            System.out.println(Math.abs(random.nextInt()));
        }

        System.out.println(Math.abs(-9)); // 9

        System.out.println(Math.round(-9.2)); // -9
        System.out.println(Math.round(-9.5)); // -9
        System.out.println(Math.round(-9.8)); // -10
        System.out.println(Math.round(9.2)); // 9
        System.out.println(Math.round(9.5)); // 10
        System.out.println(Math.round(9.8)); // 10
    }
}
```
输出:

Math类中包含了很多数学工具方法。各方法都是static，所以不用创建Math实例则可以直接通过类名调用。
Match类的文档：[https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Math.html](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Math.html)


### 2.13 String和StringBuilder
- **String**

String不可变，String用来存储字符的数据是private的，而且不提供任何修改内容的方法，所以String对象一旦生成，内容就是完全不可能被修改的。

String类的文档：[https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html)

- **StringBuiler**

StringBuilder首先是可变的
```java
package com.geekbang.learn;

public class LearnStringBuilder {

    public static void main(String[] args) {

        // TODO StringBuilder首先是可变的
        // TODO 而且对它进行操作的方法，都会返回this自引用。这样我们就可以一直点下去，对String进行构造。
        StringBuilder strBuilder = new StringBuilder();

        long longVal = 123456789;

        strBuilder.append(true).append("abc").append(longVal);

        System.out.println(strBuilder.toString());
        System.out.println(strBuilder.reverse().toString());
        System.out.println(strBuilder.reverse().toString());
        System.out.println(strBuilder.toString());

        System.out.println(strBuilder.delete(0, 4).toString());

        System.out.println(strBuilder.insert(3,"LLLLL").toString());


    }

}
```
输出：
```
trueabc123456789
987654321cbaeurt
trueabc123456789
trueabc123456789
abc123456789
abcLLLLL123456789
```

### 2.23 Class类
Class类是代表类的类，每个Class类的实例，都代表一个类。通过Class中的方法可以获取对象中一些类的信息。

**示例程序：**
```java
package com.geekbang;

import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;
import com.geekbang.supermarket.ShellColorChangePhone;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ClassOfClassAppMain {


    public static void main(String... args) throws NoSuchFieldException, NoSuchMethodException {
        LittleSuperMarket superMarket = new LittleSuperMarket("大卖场",
                "世纪大道1号", 500, 600, 100);

        MerchandiseV2 m100 = superMarket.getMerchandiseOf(100);

        // >> TODO
        // Object类里的getClass方法，可以得到
        // TODO 另一种获得Class实例的方法，直接类名点
//        Class clazz = ShellColorChangePhone.class;
         Class clazz = m100.getClass();

        System.out.println(clazz.getName());  // com.geekbang.supermarket.ShellColorChangePhone
        System.out.println(clazz.getSimpleName()); // ShellColorChangePhone

        // TODO 通过一个类的Class实例，可以获取一个类所有的信息，包括成员变量，方法，等
        Field countField = clazz.getField("count");
        Field nameField = clazz.getField("count");
        // countField: (public int com.geekbang.supermarket.MerchandiseV2.count) nameField: (public int com.geekbang.supermarket.MerchandiseV2.count)
        System.out.println("countField: (" + countField.toString() + ") nameField: (" + nameField.toString() + ")"); 

        // >> TODO 变长参数和它的等价形式
        Method equalsMethod = clazz.getMethod("equals", Object.class);
        Method buyMethod = clazz.getMethod("buy", int.class);

        //equalsMethod: (public boolean com.geekbang.supermarket.MerchandiseV2.equals(java.lang.Object)) buyMethod: (public double com.geekbang.supermarket.Phone.buy(int))
        System.out.println("equalsMethod: (" + equalsMethod.toString() + ") buyMethod: (" + buyMethod.toString() + ")");

    }

}
```

### 2.25 枚举（enum）
枚举的定义：

```java
package com.geekbang.supermarket;

// >> TODO 使用enum而非class声明
public enum Category {

    // >> TODO 必须在开始的时候以这种形式，创建所有的枚举对象
    FOOD(1),
    // >> TODO 不可以重名
//    FOOD(1),
    COOK(3),
    SNACK(5),
    CLOTHES(7),
    ELECTRIC(9);

    // 可以有属性
    private int id;

    // >> TODO 构造方法必须是private的，不写也是private的
    Category(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

//    public void setId(int id) {
//        this.id = id;
//    }


    @Override
    public String toString() {
        return "Category{" +
            "id=" + id +
            '}';
    }

}
```
enum类型不再继承Object类，而是默认继承的Enum类，其中的一些方法都可以使用。**枚举的使用：**
```java
package com.geekbang;

import com.geekbang.supermarket.Category;

import java.util.Scanner;

public class UseEnum {
    public static void main(String[] args) {
        // >> TODO 获取所有枚举，看看枚举实例有哪些方法
        for (Category category : Category.values()) {
            System.out.println("-----------" + category.getId() + "------------");
            System.out.println(category.ordinal());
            System.out.println(category.name());
            System.out.println(category.toString());
        }

        System.out.println();
        // >> TODO 根据名字获取枚举
        System.out.println(Category.valueOf("FOOD"));
//        System.out.println(Category.valueOf("food"));  //没有这个枚举，会报错

        Scanner in = new Scanner(System.in);
        System.out.println("请输入枚举的名字：");
        String categoryName = in.next();
        Category enumInput = Category.valueOf(categoryName.trim().toUpperCase());
        System.out.println("枚举的信息：" + enumInput.toString());

        System.out.println("请输入要比较的枚举的名字：");
        String categoryName2 = in.next();
        Category enumInput2 = Category.valueOf(categoryName2.trim().toUpperCase());
        System.out.println("第二次输入的枚举的信息：" + enumInput2.toString());

        System.out.println(enumInput == enumInput2);  //枚举在定义时候就已经创建了实例
    }
}

```


### 2.29 各种特殊类
#### 2.29.1 静态内部类

1. 静态内部类，是在类中使用static修饰的类
2. 静态内部类，可以有访问控制符。静态内部类和静态方法，静态变量一样，都是类的静态组成部分
3. 静态内部类也是类，在继承，实现接口方面，都是一样的。
4. 静态内部类是静态的，就好像静态方法一样，没有this自引用，可以通过引用访问Phone对象的private属性
5. 静态内部类，可以认为是和静态方法、静态成员一样，是类的一个类内部的一个成员，一个组成部分
6. 使用的时候，也可以点出来，只是点出来的是个类而已。点出来以后，改怎么用一个类都行

```java
// 直接使用静态内部类
Phone.CPU cpu = new Phone.CPU(5.5, "default");
```

**示例：**

```java
package com.geekbang.supermarket;

public class Phone extends MerchandiseV2 {

    // 给Phone增加新的属性和方法
    private double screenSize;
    private CPU cpu;
    private int memoryG;
    private int storageG;
    private String brand;
    private String os;

    // >> TODO 静态内部类，是在类中使用static修饰的类
    // >> TODO 静态内部类，可以有访问控制符。静态内部类和静态方法，静态变量一样，都是类的静态组成部分
    // >> TODO 静态内部类也是类，在继承，实现接口方面，都是一样的。以后我们讲的类，不特殊说明，在这方面都是一样的
    public static class CPU {
        private double speed;
        private String producer;

        public CPU(double speed, String producer) {
            this.speed = speed;
            this.producer = producer;
        }

        public double getSpeed() {
            // >> TODO 静态内部类，代码和这个类本身的访问权限一样，可以访问外部（Phone）的private属性
            // >> TODO 注意，这并不少说它可以访问private变量，
            // >> TODO 静态内部类是静态的，就好像静态方法一样，没有this自引用，可以通过引用访问Phone对象的private属性
            // 仅作演示访问性，不具有实际意义
            Phone phone = null;
            phone.memoryG = 99;
            return speed;
        }

        public void setSpeed(double speed) {
            this.speed = speed;
        }

        public String getProducer() {
            return producer;
        }

        public void setProducer(String producer) {
            this.producer = producer;
        }

        //重载了toString方法
        @Override
        public String toString() {
            return "CPU{" +
                "speed=" + speed +
                ", producer='" + producer + '\'' +
                '}';
        }

        // >> TODO 静态内部类，里面可以有任意合法的类的组成部分，包括静态内部类
//        public static class ABC{
//
//        }

    }

    public void accessStaticClass(){
        // >> TODO 同样，外部类也可以访问静态内部类（CPU）的private属性
        // 仅作演示访问性，不具有实际意义
        this.cpu.producer = "";
    }


    public Phone(
        String name, String id, int count, double soldPrice, double purchasePrice,
        double screenSize, double cpuHZ, int memoryG, int storageG, String brand, String os
    ) {

        this.screenSize = screenSize;
        // >> TODO 可以像平常的类一样使用静态内部类
        this.cpu = new CPU(cpuHZ, "Default");
        this.memoryG = memoryG;
        this.storageG = storageG;
        this.brand = brand;
        this.os = os;

        this.setName(name);
        this.setId(id);
        this.setCount(count);
        this.setSoldPrice(soldPrice);
        this.setPurchasePrice(purchasePrice);
    }

    public void describePhone() {

        System.out.println("此手机商品属性如下");
        describe();
        System.out.println("手机厂商为" + brand + "；系统为" + os + "；硬件配置如下：\n" +
            "屏幕：" + screenSize + "寸\n" +
            "cpu信息：" + cpu + " \n" +
            "内存" + memoryG + "Gb\n" +
            "存储空间" + storageG + "Gb\n");

    }

}

// >> TODO 没有修饰符，类名可以与文件名不同，非共有类和静态内部类，实际区别就在于能否访问类的private成员
class Memory {
    private long capacity;
    private String producer;

    public Memory(long capacity, String producer) {
        this.capacity = capacity;
        this.producer = producer;
    }

    public void test(){
        // >> TODO 在类的外面的代码，不能访问类的private成员
        // 仅作演示访问性，不具有实际意义
//        Phone ph = null;
//        ph.screenSize = 9;
    }

    public long getCapacity() {
        return capacity;
    }

    public void setCapacity(long capacity) {
        this.capacity = capacity;
    }

    public String getProducer() {
        return producer;
    }

    public void setProducer(String producer) {
        this.producer = producer;
    }
}

```

#### 2.29.2 成员内部类

1. 成员内部类，是在类中直接定义类
2. 不可以包含任何静态的成分，比如静态方法，静态变量，静态内部类。否则会造成内外部类初始化问题。
3. 可以有访问控制符。成员内部类和成员方法，成员变量一样，都是类的组成部分
4. 可以有final static的基本数据类型变量
5. 成员内部类，代码和这个类本身的访问权限一样，可以访问外部（Phone）的private属性
6. 成员内部类中有一个外部类的引用，其访问外部类的对象的成员属性就是使用这个引用，完整写法是：类名.this.属性/方法
7. 成员内部类，里面可以有任意合法的类的组成部分，包括成员内部类，但是不可以有静态内部类
8. 外部类也可以访问成员内部类（CPU）的private属性
9. 可以像平常的类一样使用成员内部类

**外部使用成员内部类：**
```java
Phone.CPU cpu = phone.new CPU("default");
```

**示例：**
```java
package com.geekbang.supermarket;

public class Phone extends MerchandiseV2 {

    // 给Phone增加新的属性和方法
    private double screenSize;
    private CPU cpu;
    private Memory memory;
    private int storageG;
    private String brand;
    private String os;
    private double speed;
    private int memoryG;

    // >> TODO 成员内部类，是在类中直接定义类
    // >> TODO 成员内部类，不可以包含任何静态的成分，比如静态方法，静态变量，静态内部类。否则会造成内外部类初始化问题。
    // >> TODO 成员内部类，可以有访问控制符。成员内部类和成员方法，成员变量一样，都是类的组成部分
    public class CPU {
        // >> TODO 可以有final static的基本数据类型变量
        final static int abc = 999;

        private String producer;

        public CPU(String producer) {
            this.producer = producer;
        }

        public double getSpeed() {
            // >> TODO 成员内部类，代码和这个类本身的访问权限一样，可以访问外部（Phone）的private属性
            // >> TODO 成员内部类中有一个外部类的引用，其访问外部类的对象的成员属性就是使用这个引用，完整写法是：类名.this.属性/方法
            return Phone.this.speed;
        }

        public String getProducer() {
            return producer;
        }

        public void setProducer(String producer) {
            this.producer = producer;
        }

        @Override
        public String toString() {
            return "CPU{" +
                "speed=" + getSpeed() +
                ", producer='" + producer + '\'' +
                '}';
        }

        // >> TODO 成员内部类，里面可以有任意合法的类的组成部分，包括成员内部类，但是不可以有静态内部类
//        public class ABC{
//            public void test(){
//
//            }
//        }

    }

    public void accessStaticClass() {
        // >> TODO 同样，外部类也可以访问成员内部类（CPU）的private属性
        // 仅作演示访问性，不具有实际意义
        this.cpu.producer = "";
    }


    public Phone(
        String name, String id, int count, double soldPrice, double purchasePrice,
        double screenSize, double cpuHZ, int memoryG, int storageG, String brand, String os
    ) {
        this.screenSize = screenSize;
        // >> TODO 可以像平常的类一样使用成员内部类
        this.speed = cpuHZ;
        this.memoryG = memoryG;
        this.cpu = new CPU("Default");
        this.memory = new Memory("Default");
        this.storageG = storageG;
        this.brand = brand;
        this.os = os;

        this.setName(name);
        this.setId(id);
        this.setCount(count);
        this.setSoldPrice(soldPrice);
        this.setPurchasePrice(purchasePrice);
    }

    public void describePhone() {
        System.out.println("此手机商品属性如下");
        describe();
        System.out.println("手机厂商为" + brand + "；系统为" + os + "；硬件配置如下：\n" +
            "屏幕：" + screenSize + "寸\n" +
            "cpu信息：" + cpu + " \n" +
            "内存信息：" + memory + "\n" +
            "存储空间" + storageG + "Gb\n");
    }
}

```

#### 2.29.3 局部内部类

1. 局部内部类，是在类中直接定义类
2. 局部内部类，不可以包含任何静态的成分，比如静态方法，静态变量，静态内部类。否则会造成内外部类初始化问题。
3. 局部内部类，不可以有访问控制符。局部内部类和成员方法，成员变量一样，都是类的组成部分
4. 可以有final static的基本数据类型变量
5. 局部内部类，代码和这个类本身的访问权限一样
6. 局部内部类中有一个外部类的引用
7. 局部内部类访问外部类的对象的成员属性的完整写法如下，类名.this.属性/方法
8. 局部内部类中无法修改传进去的参数，定义的变量都是final

**示例：**
```java
package com.geekbang.supermarket;

public class Phone extends MerchandiseV2 {

    // 给Phone增加新的属性和方法
    private double screenSize;
    private UnitSpec cpu;
    private UnitSpec memoryG;
    private int storageG;
    private String brand;
    private String os;
    private double speed;

    // >> TODO 接口也可以定义为静态内部接口，但是一般不这么做。接口的目的是为了让更多人实现，所以一般会是单独一个文件作为公共接口
    public static interface UnitSpec {
        public double getNumSpec();

        public String getProducer();
    }

    public Phone(
        String name, String id, int count, double soldPrice, double purchasePrice,
        double screenSize, double cpuHZ, int memoryG, int storageG, String brand, String os
    ) {

        double localCPUHZ = cpuHZ;

//        localCPUHZ = Math.random();

        // >> TODO 局部内部类，是在类中直接定义类
        // >> TODO 局部内部类，不可以包含任何静态的成分，比如静态方法，静态变量，静态内部类。否则会造成内外部类初始化问题。
        // >> TODO 局部内部类，不可以有访问控制符。局部内部类和成员方法，成员变量一样，都是类的组成部分
        class CPU implements UnitSpec {
            // >> TODO 可以有final static的基本数据类型变量
            final static int abc = 999;

            private String producer;

            public CPU(String producer) {
                this.producer = producer;
            }

            public double getNumSpec() {
                // >> TODO 局部内部类，代码和这个类本身的访问权限一样，可以访问外部（Phone）的private属性
                // >> TODO 局部内部类中有一个外部类的引用
                // >> TODO 局部内部类访问外部类的对象的成员属性的完整写法如下，类名.this.属性/方法
                // >> TODO 以上都和成员内部类一样。除此之外，局部内部类还可以访问参数和局部变量，但是它俩必须是实际final的
              // 仅做访问数据的演示，没有实际意义
                return Math.max(Phone.this.speed, Math.max(cpuHZ, localCPUHZ));
            }

            public String getProducer() {
                return producer;
            }

            public void setProducer(String producer) {
                this.producer = producer;
            }

            @Override
            public String toString() {
                return "CPU{" +
                    "speed=" + getNumSpec() +
                    ", producer='" + producer + '\'' +
                    '}';
            }

            // >> TODO 局部内部类，就好像局部变量一样，方法内部的东西出了代码就不可被访问，
            // >> TODO 所以可以再定义类，但是不能有访问控制符，也不能是static，就好像成员变量没有访问控制符没有static一样
            // >> TODO 但是可以有final。记忆要点：和局部变量一样
//            final class ABC {
//                public void test() {
//
//                }
//            }

        }


        class Memory implements UnitSpec {
            private String producer;

            public Memory(String producer) {
                this.producer = producer;
            }

            @Override
            public double getNumSpec() {
                return memoryG;
            }

            public String getProducer() {
                return producer;
            }

            public String toString() {
                return "Memory{" +
                    "storage=" + getNumSpec() +
                    ", producer='" + producer + '\'' +
                    '}';
            }

        }

        this.screenSize = screenSize;
        // >> TODO 可以像平常的类一样使用局部内部类
        this.speed = cpuHZ;
        this.cpu = new CPU("Default");
        this.memoryG = new Memory("Default");
        this.storageG = storageG;
        this.brand = brand;
        this.os = os;

        this.setName(name);
        this.setId(id);
        this.setCount(count);
        this.setSoldPrice(soldPrice);
        this.setPurchasePrice(purchasePrice);
    }

    public void describePhone() {
        System.out.println("此手机商品属性如下");
        describe();
        System.out.println("手机厂商为" + brand + "；系统为" + os + "；硬件配置如下：\n" +
            "屏幕：" + screenSize + "寸\n" +
            "cpu信息：" + cpu + " \n" +
            "内存" + memoryG.getNumSpec() + "Gb\n" +
            "存储空间" + storageG + "Gb\n");
    }

}

```

#### 2.29.4 匿名类
方法里的匿名类在访问局部变量和参数时，它们也必须是实际final的
匿名类的实例作为参数传递也没问题

```java
// >> TODO 匿名类的语法如下，new后面跟着一个接口或者抽象类
    private UnitSpec anywhere = new UnitSpec() {
        @Override
        public double getNumSpec() {
            return Phone.this.speed;
        }

        @Override
        public String getProducer() {
            return "Here";
        }
    };

 // >> TODO 对于抽象类，也可以给构造方法传递参数
    private UnitSpecAbs anywhereAbs = new UnitSpecAbs(1.2, "default") {
        @Override
        public double getNumSpec() {
            return Math.max(Phone.this.speed, this.getSpec());
        }

        @Override
        public String getProducer() {
            return this.getProducerStr();
        }
    };
```

#### 2.29.5 特殊类的总结
- 枚举

1. 枚举就是有固定个数实例的类
2. 枚举的父类是Enum

- 非公共类

1. 就是被缺省访问控制符修饰的类，和public class的区别仅仅是可以被访问的范围不一样
2. 如果一个文件只有非公有类，那么类名和文件名可以不一样

- 内部类

1. 内部类被认为是类本身的代码，外部类的private成员对其可见
2. 类里面可以有静态变量，成员变量和局部变量。内部类可以分为如下三种：

> 1. 静态内部类：可以有访问修饰符，可以在类外部访问（对比静态变量）
> 2. 成员内部类：可以有访问修饰符，有外部类对象的this自引用（对比成员方法），可以在外部使用，但是创建对象语法需要指明外部对象
> 3. 局部内部类：没有访问修饰符（对比局部变量），有外部的引用，访问参数和局部变量，必须是final的。

- 匿名类

1. 匿名类是一种创建接口和抽象类对象的语法，任何可以new一个对象的地方，都可以使用匿名类。
2. 匿名类只能实现/继承一个接口/抽象类，本身没有名字
3. 如果是在成员方法或者给成员方法赋值时创建匿名类，那么会有外部对象的this自引用
4. 匿名类也可以访问外部类的private属性

> 无论时内部类还是匿名类，类都是只有一个，对象可以有多个，不会每次执行到内部类声明的地方，就创建一个新的类。


# 9 UML类图
- UML(Unified Modeling Language，统一建模语言)，用来描述 软件模型 和 架构 的图形化语言。
- 常用的UML工具软件有 PowerDesinger、Rose和 Enterprise Architect。
- UML工具软件不仅可以绘制软件开发中所需的各种图表，还可以生成对应的源代码
- +表示 public 类型， - 表示 private 类型，#表示protected类型
- 方法的写法: 方法的类型(+、-) 方法名(参数名: 参数类型):返回值类型
- 斜体表示抽象方法或类。
![image.png](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240701223715.png)

# 10 native关键字的理解
使用native关键字说明这个方法是原生函数，也就是这个方法是用`C/C++`等非Java语言实现的，并且`被编译成了DLL`，由Java去调用。
- 本地方法是有方法体的，用c语言编写。由于本地方法的方法体源码没有对我们开源，所以我们看不到方法体
- 在Java中定义一个native方法时，并不提供实现体。

**1. 为什么要用native方法**
Java使用起来非常方便，然而有些层次的任务用java实现起来不容易，或者我们对程序的效率很在意时，例如：Java需要与一些底层操作系统或某些硬件交换信息时的情况。native方法正是这样一种交流机制：它为我们提供了一个非常简洁的接口，而且我们无需去了解Java应用之外的繁琐的细节。

**2. native声明的方法，对于调用者，可以当做和其他Java方法一样使用**
native method的存在并不会对其他类调用这些本地方法产生任何影响，实际上调用这些方法的其他类甚至不知道它所调用的是一个本地方法。JVM将控制调用本地方法的所有细节。
