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
## 2 Java面向对象编程
### 2.1 类

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

### 2.2 引用类型
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

### 2.3 类对象和引用的关系
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

### 2.4 引用的缺省值--null
1. null是引用类型的缺省值
2. null代表空，不存在。
3. 引用类型的数组创建出来，初始值都是空

- null带来的问题

1. 大名鼎鼎的NullPointerException（NPE）
2. 如果不确定，使用前先判断引用是不是空

### 2.5 包和访问修饰符

* 为了避免类在一起混乱，可以把类放在文件夹里。这时就需要用 package 语句告诉Java 这个类在哪个 package 里。 package 语句要和源文件的目录完全对应，大小写要一致
*  package 读作包。一般来说，类都会在包里，而不会直接放在根目录
* 不同的包里可以有相同名字的类
* 一个类只能有一个 package 语句，如果有 package 语句，则必须是类的第一行有效代码

1. 使用import

每次使用都带包名很繁琐， 可以在使用的类的上面使用 import 语句， 一次性解决问题， 就可以直接使用类了。 就好像我们之前用过的 Scanner类。

```java
import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;
```

2. 属性访问修饰符：public

* 被 public 修饰的属性，可以被任意包中的类访问
* 没有访问修饰符的属性，称作缺省的访问修饰符，可以被本包内的其他类和自己的对象
* 访问修饰符是一种限制或者允许属性访问的修饰符

3. 类的全限定名

* 包名 + 类名 = 类的全限定名。也可以简称为类的全名
* 同一个 Java 程序中全限定名字不可重复

### 2.6 隐藏this的自引用
通过this自引用访问类的成员变量，this指向的地址也是类的实例的地址。


### 2.7 方法的签名和重载

* 方法签名 : 方法名+依次参数类型。注意，返回值不属于方法签名。方法签名是一个方法在一个类中的唯一标识
* 同一个类中方法可以重名，但是签名不可以重复。一个类中如果定义了名字相同，签名不同的方法，就叫做方法的重载
* 看代码: 重写我们的购买方法，理解方法签

1. 重载的参数匹配规则

如果有如下重载方法，在java中的自动类型转换匹配逻辑为：
```java
public double buy(int count) {}
public double buy(double count){}
```
如果传的参数是 byte, short, int,类型会调用buy(int count)方法，如果是long, float, double 类型会调用buy(double count)方法。

> 无论是否重载参数类型可以不完全匹配的规则是"实参数可以自动类型转换成形参类型"
重载的特殊之处是，参数满足自动自动类型转换的方法有好几个，重载的规则是选择最"近"的去调用

### 2.8 构造方法

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

### 2.9 静态变量和静态方法
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

### 2.10 static代码块和static变量
**示例：**
```java
package com.geekbang.supermarket;

public class DiscountMgr {

    public static void main(String[] args) {
        System.out.println("最终main 方法中使用的SVIP_DISCOUNT是" + SVIP_DISCOUNT);
    }

    public static double BASE_DISCOUNT;

    public static double VIP_DISCOUNT;

    // >> TODO 使用某个静态变量的代码块必须在静态变量后面
    public static double SVIP_DISCOUNT;


    static {
        BASE_DISCOUNT = 0.99;
        VIP_DISCOUNT = 0.85;
        SVIP_DISCOUNT = 0.75;

        // >> TODO 静态代码块里当然可以有任意的合法代码
        System.out.println("静态代码块1里的SVIP_DISCOUNT" + SVIP_DISCOUNT);

        // >> TODO 这段代码在哪个方法中呢？<clinit>，即class init。会在每个class初始化的时候被调用一次
//         SVIP_DISCOUNT = 9/0;
    }

    // >> TODO 其实给静态变量赋值也是放在代码块里的，static代码块可以有多个，是从上向下顺序执行的。
    //    TODO 可以认为这些代码都被组织到了一个clinit方法里
    // public static double WHERE_AM_I = 9/0;

//     public static double SVIP_DISCOUNT;

    static {
        SVIP_DISCOUNT = 0.1;
        System.out.println("静态代码块2里的SVIP_DISCOUNT" + SVIP_DISCOUNT);
    }


}
```
输出：
```
静态代码块1里的SVIP_DISCOUNT0.75
静态代码块2里的SVIP_DISCOUNT0.1
最终main 方法中使用的SVIP_DISCOUNT是0.1
```

### 2.11 方法和属性的可见性修饰符
- 可见性修饰符用在类、 成员方法、 构造方法、 静态方法和属性上， 其可见性的范围是一样的
- 可见性修饰符：
    - public：全局可见
    - 缺省：当前包可见
    - private：当前类可见
- 成员变量应该是private的，不需要让外部使用的方法应该都是private的。

```java
package com.geekbang.supermarket;

// >> TODO 类，静态方法，静态变量，成员变量，构造方法，成员方法都可以使用访问修饰符
public class MerchandiseV2 {

    // >> TODO 成员变量应该都声明为private
    // >> TODO 如果要读写这些成员变量，最好使用get set方法，这些方法应该是public的
    // >> TODO 这样做的好处是，如果有需要，可以通过代码，检查每个属性值是否合法。
    private String name;
    private String id;
    private int count;
    private double soldPrice;
    private double purchasePrice;
    private NonPublicClassCanUseAnyName nonPublicClassCanUseAnyName;
    public static double DISCOUNT = 0.1;

    // >> TODO 构造方法如果是private的，那么就只有当前的类可以调用这个构造方法
    public MerchandiseV2(String name, String id, int count, double soldPrice, double purchasePrice) {
        this.name = name;
        this.id = id;
        this.count = count;
        this.soldPrice = soldPrice;
        this.purchasePrice = purchasePrice;
        // soldPrice = 9/0;
    }

    // >> TODO 有些时候，会把所有的构造方法都定义成private的，然后使用静态方法调用构造方法
    // >> TODO 同样的，这样的好处是可以通过代码，检查每个属性值是否合法。
    public static MerchandiseV2 createMerchandise(String name, String id, int count,
                                                  double soldPrice, double purchasePrice) {
        if (soldPrice < 0 || purchasePrice < 0) {
            return null;
        }
        return new MerchandiseV2(name, id, count, soldPrice, purchasePrice);
    }

    public MerchandiseV2(String name, String id, int count, double soldPrice) {
        this(name, id, count, soldPrice, soldPrice * 0.8);
    }

    public MerchandiseV2() {
        this("无名", "000", 0, 1, 1.1);
    }

    // >> TODO public的方法类似一种约定，既然外面的代码可以使用，就意味着不能乱改。比如签名不能改之类的
    public void describe() {
        System.out.println("商品名字叫做" + name + "，id是" + id + "。 商品售价是" + soldPrice
            + "。商品进价是" + purchasePrice + "。商品库存量是" + count +
            "。销售一个的毛利润是" + (soldPrice - purchasePrice));
        freeStyle();
    }

    // >> TODO 对于private的方法，因为类外面掉不到，所以无论怎么改，也不会影响（直接影响）类外面的代码
    private void freeStyle() {

    }

    public double calculateProfit() {
        double profit = soldPrice - purchasePrice;
        return profit;
    }

    public double buy(int count) {
        if (this.count < count) {
            return -1;
        }
        return this.count -= count;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public double getSoldPrice() {
        return soldPrice;
    }

    public void setSoldPrice(double soldPrice) {
        this.soldPrice = soldPrice;
    }

    public double getPurchasePrice() {
        return purchasePrice;
    }

    public void setPurchasePrice(double purchasePrice) {
        this.purchasePrice = purchasePrice;
    }
}

```
- 非public的类，类名可以和文件名不相同
- 非public的类不能在包外使用，可以被包内的其他类使用


### 2.12 Math和Scanner的使用
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

### 2.14 main方法和System类
1. main方法

- main方法只是一个静态的，有String[]做参数的，没有返回值的方法而已，Java可以把main方法作为程序入口。
- 通过执行程序给main方法传递参数

2. System类

- System类中有很多和系统相关的方法，用的最多的是in和out来读取和输出数据
- System的文档：[https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html)

```java
package com.geekbang.learn;

public class LearnSystem {
    public static void main(String[] args) {
        long startMS = System.currentTimeMillis();

        int counter = 0;
        for (int i = 0; i < 1000; i++) {
            counter++;
        }
        //获取当前时间，单位毫秒
        long endMS = System.currentTimeMillis();
        System.out.println("程序执行使用了几个毫秒？" + (endMS - startMS));

        //获取当前时间，单位纳秒
        long startNS = System.nanoTime();
        counter = 0;
        for (int i = 0; i < 1000; i++) {
            counter++;
        }

        long endNS = System.nanoTime();
        System.out.println("程序执行使用了几个纳秒？" + (endNS - startNS));
    }
}

```

### 2.15 继承
1. 继承

- 子类继承了父类的方法和属性
- 使用子类的引用可以调用父类的共有方法
- 使用子类的引用可以访问父类的共有属性
- 就好像子类的引用可以一物二用，既可以当作父类的引用使用，又可以当作子类的引用使用

* 继承的语法就是在类名后面使用extends 加 要继承的类名
* Java中只允许一个类有一个直接的父类（Parent Class），即所谓的单继承
* 子类继承了父类什么呢？所有的属性和方法。

但是子类并不能访问父类的private的成员（包括方法和属性）。

2. 组合和继承

* 继承，其实表达的是一种"is-a"的关系，也就是说，在你用类构造的世界中，"子类是父类的一种特殊类别"
* 继承不是组合，继承也不只是为了能简单的拿来父类的属性和方法。如果仅仅如此，原封不动的拿来主义，组合也能做到。
* 继承也不是通过组合的方式来实现的。和组合相比，继承更像是"融合"

3. 覆盖

覆盖才是继承的精髓和终极奥义，可以通过覆盖父类的方法，重新实现子类的逻辑。

通过使用和父类方法签名一样，而且返回值也必须一样的方法，可以让子类覆盖(override)掉父类的方法

4. super：和父类对象沟通的桥梁

子类对象里可以认为有一个特殊的父类对象，这个父类对象和子类对象之间通过super关键字来沟通。

5. 继承里的静态方法：

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

### 2.16 父类和子类的引用赋值关系
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

### 2.17 多态：到底调用那个方法

* 可以调用那些方法，取决于引用类型，具体调用那个方法，取决于实例所属的类是什么。
* 覆盖是多态里最重要的一种形式。

* 无论一个方法是使用哪个引用被调用的，"它都是在实际的对象上执行的"。执行的任何一个方法，都是这个对象所属的类的方法。
* 如果没有，就去父类找，再没有，就去父类的父类找，依次寻找，知道找到。
* 换个角度理解。我们一直说子类里又一个（特殊的）父类的对象。这时候，这个特殊的父类的对象里的this自引用，是子类的引用。那么自然的，即使是在继承自父类的代码里，去调用一个方法，也是先从子类开始，一层层继承关系的找。
* 这也是Java选择单继承的重要原因之一。在多继承的情况下，如果使用不当，多态可能会非常复杂，以至于使用的代价超过其带来的好处。

### 2.18 多态：语法点
面向对象三要素：封装、继承和多态

1. 静态多态：重载

* 重载调用那个方法和参数引用类型相关
* 如果在重载的传参时进行了强制类型转换，则还是调用强制转换的类型。
* 如果传的引用类型没有完全匹配的重载函数，则会根据引用类型的继承关系，沿着当前类型向其父类找。


2. 动态多态：覆盖

程序的执行就是找到要执行的代码，并且知道执行的代码能访问那些数据，数据从哪里来。
多态的核心问题就是：要调用那个类的那个方法，这个方法用到的数据（this引用）是谁。

> 在方法中使用this自引用时，要确定调用当前方法的对象到底是哪个，其不一定是这个方法所属的类，也可能是这个类的子类对象。这是用这个this自引用的则是子类的对象。

### 2.19 instanceof操作符
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
### 2.20 protected修饰符

1. protected修饰的属性和方法，只有本包和子类可见，不同包里不可以访问protected。
2. 子类覆盖父类的方法，不可以用可见性更低的修饰符，但是可以用更高的修饰符，比如：父类的方法使用protected修饰，子类就不可以使用可见性更低的private修饰覆盖的方法。
3. 构造方法可以是protected，但是如果是private，子类就不可以覆盖了。
4. 如果父类只有一个private的构造方法，相当于这个类不能有子类

### 2.21 final 修饰符
- final修饰类：不可被继承
- final修饰方法：不可被子类覆盖

1. 构造方法不能用final修饰
2. 使用final修饰方法的形参后，则无法在方法内修改形参变量

- final修饰变量：不可被赋值

1. 修饰静态变量：需要初始化，或者在static块中初始化，并且初始化后无法修改。

```java
private final static int MAX_BUY_ONE_ORDER = 9;
//或   选其一
static {
    MAX_BUY_ONE_ORDER = 1;
}

```
2. 修饰属性：只能在定义时初始化或者构造方法中初始化，初始化后无法修改。
3. 修饰引用：引用类型本身初始化后无法修改，但可以通过引用修改引用的对象内容。
```java
int[] array1 = new int[10];
final int[] array = new int[10];
array = array1;  //报错，无法修改final变量
for(int b : array) {  //for循环的一种新形式
    System.out.println(b);
}
```
4. 修饰局部变量：只能初始化一次，后边不能再次修改

### 2.22 Object类
所有的类，都直接或间接地继承自Object类。
Object类的对象可以指向任意类，Object类中没有属性，只有方法，定义的对象如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202308081038318.png)

下边介绍一些Object类中的方法：
1. **equals**

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
但是在定义的字符串很长时，则会突破这个优化，如上，输入了一个很长的字符串，使用"=="比较是不相同的，但是使用equals是相同的，下边看下String类中对equals方法的覆盖实现：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202308081053010.png)
String中重新实现了equals方法按字符进行遍历比较，所以再长的字符串的情况下依然可以比较。

2. **hashCode**

hashCode可以翻译为哈希码，或者散列码。应该是一个表示对象的特征的int整数。
自定义类中可以进行覆盖实现，来表示依次来表示类的唯一标识。

> equals和hashCode是最常覆盖的两个方法，实现逻辑为，equals方法为true，则hashCode就相等，但hashCode相等，equals不一定为true。

3. **toString**

toString方法默认也是存在在所有类中，在部分场景下默认会被调用。
```java
package com.geekbang;

import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;

public class ToStringAppMain {
    public static void main(String[] args) {
        LittleSuperMarket superMarket = new LittleSuperMarket("大卖场",
            "世纪大道1号", 500, 600, 100);

        MerchandiseV2 m100 = superMarket.getMerchandiseOf(100);

        StringBuilder strBuilder = new StringBuilder();

        //append会调用m100中实现的toString方法
        strBuilder.append("商品100是：").append(m100);

        // >> TODO 因为toString是Object里的方法，所以任何一个Java的对象，都可以调用这个方法
        // >> TODO 内容好像不大全，补充一下？
        System.out.println(strBuilder.toString());
        System.out.println(m100);

    }

}
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

### 2.24 反射
- 使用反射访问属性
- 使用反射访问方法
- 使用反射访问静态方法和属性
- 通过反射访问 private 的方法和属性

**示例：**
```java
package com.geekbang;

import com.geekbang.supermarket.LittleSuperMarket;
import com.geekbang.supermarket.MerchandiseV2;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ReflectionAppMain {
    public static void main(String... args) throws NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        LittleSuperMarket superMarket = new LittleSuperMarket("大卖场",
                "世纪大道1号", 500, 600, 100);

        MerchandiseV2 m100 = superMarket.getMerchandiseOf(100);

        // TODO 另一种获得Class实例的方法，直接类名点
        Class clazz = MerchandiseV2.class;

        //通过反射访问对象的属性和方法
        Field countField = clazz.getField("count");
        System.out.println("通过反射获取count的值："+countField.get(m100)); //通过反射获取count的值：100

        Method buyMethod = clazz.getMethod("buy", int.class);
        System.out.println(buyMethod.invoke(m100, 10));  // 购买失败，手机一次最多只能买5个


        // 针对private的属性 soldPrice，需要用如下方法获取
        Field soldPriceField = clazz.getDeclaredField("soldPrice");
        soldPriceField.setAccessible(true);  //设置属性可以修改
        System.out.println(soldPriceField.get(m100));  //1999
        soldPriceField.set(m100, 999);  // 修改为 999
        System.out.println(soldPriceField.get(m100)); // 999.0
//        System.out.println(m100.soldPrice);  //无法使用引用直接访问 private属性

        printFields(clazz);
        //访问static 属性，不再需要指定引用
        Field field = clazz.getField("STATIC_MEMBER");
        System.out.println(field.get(null));
        
        // 针对private的方法
        Method descMethod = clazz.getDeclaredMethod("describe");
        descMethod.setAccessible(true);
        descMethod.invoke(m100);
        descMethod.invoke(superMarket.getMerchandiseOf(0));
        descMethod.invoke(superMarket.getMerchandiseOf(10));
//        m100.describe();

        //访问static 方法
        Method staticMethod = clazz.getMethod("getNameOf", MerchandiseV2.class);
        String str = (String) staticMethod.invoke(null, m100);
        System.out.println(str);
        
//        public double buy(int count)
        Method buyMethod = clazz.getMethod("buy", int.class);
        buyMethod.invoke(m100, 1);
        m100.buy(10);


    }

    public static void printFields(Class clazz) {
        System.out.println(clazz.getName() + "里的field");
        for (Field field : clazz.getFields()) {  //获取类中的所有属性和类型
            System.out.println(field.getType() + " " + field.getName());
        }
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

### 2.26 接口（interface）
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


### 2.27 抽象类：接口和类的混合体

* 抽象类用abstract修饰，抽象类可以继承别的类或者抽象类，也可以实现接口
* 抽象类可以有抽象方法，抽象方法可以来自实现的接口，也可以自己定义
* 抽象类不可以被实例化
* 抽象类也可以没有抽象方法，没有抽象方法的抽象类，也不可以被实例化。
* 简单来说，抽象类就两点特殊：1）被abstract修饰，可以有抽象方法 2）不可以被实例化

**抽象类的定义：**
```java
package com.geekbang.supermarket;

import java.util.Date;

public abstract class AbstractExpireDateMerchandise extends MerchandiseV2 implements ExpireDateMerchandise {

    private Date produceDate;
    private Date expirationDate;

    // >> TODO 抽象类里构造方法的语法和类一样。
    public AbstractExpireDateMerchandise(String name, String id, int count, double soldPrice, double purchasePrice, Date produceDate, Date expirationDate) {
        super(name, id, count, soldPrice, purchasePrice);
        this.produceDate = produceDate;
        this.expirationDate = expirationDate;
    }

    public AbstractExpireDateMerchandise(String name, String id, int count, double soldPrice, Date produceDate, Date expirationDate) {
        super(name, id, count, soldPrice);
        this.produceDate = produceDate;
        this.expirationDate = expirationDate;
    }

    public AbstractExpireDateMerchandise(Date produceDate, Date expirationDate) {
        this.produceDate = produceDate;
        this.expirationDate = expirationDate;
    }

    // >> TODO @ 是Java中的注解（annotation），后面我们会详细讲述
    // >> TODO @Override代表此方法覆盖了父类的方法/实现了继承的接口的方法，否则会报错
    public boolean notExpireInDays(int days) {
        return daysBeforeExpire() > 0;
    }

    public Date getProducedDate() {
        return produceDate;
    }

    public Date getExpireDate() {
        return expirationDate;
    }

    public double leftDatePercentage() {
        return 1.0 * daysBeforeExpire() / (daysBeforeExpire() + daysAfterProduce());
    }

//    @Override  抽象类未实现继承接口的此方法
//    public double actualValueNow(double leftDatePercentage) {
//        return 0;
//    }

    // >> TODO 抽象类里自己定义的抽象方法，可以是protected，也可以是缺省的，这点和接口不一样
//    protected abstract void test();


    // TODO 这俩方法是私有的，返回值以后即使改成int，也没有顾忌
    private long daysBeforeExpire() {
        long expireMS = expirationDate.getTime();
        long left = expireMS - System.currentTimeMillis();
        if (left < 0) {
            return -1;
        }
        // 返回值是long，是根据left的类型决定的
        return left / (24 * 3600 * 1000);
    }

    private long daysAfterProduce() {
        long produceMS = produceDate.getTime();
        long past = System.currentTimeMillis() - produceMS;
        if (past < 0) {
            // 生产日期是未来的一个时间？315电话赶紧打起来。
            return -1;
        }
        // 返回值是long，是根据left的类型决定的
        return past / (24 * 3600 * 1000);
    }
}

```
抽象类只实现了继承接口中的部分方法，然后这样继承抽象类的类就不需要实现接口中所有的方法了，可以只实现抽象类中没有实现的接口的方法。
```java
package com.geekbang.supermarket;

import java.util.Date;

// >> TODO 一个类只能继承一个父类，即使是抽象类，也只能是一个
public class GamePointCard extends AbstractExpireDateMerchandise implements VirtualMerchandise {

    public GamePointCard(String name, String id, int count, double soldPrice, double purchasePrice, Date produceDate, Date expirationDate) {
        super(name, id, count, soldPrice, purchasePrice, produceDate, expirationDate);
    }
    
    @Override
    public double actualValueNow(double leftDatePercentage) {
        return super.getSoldPrice();
    }

}

```
GamePointCard类中只实现了抽象类中没有实现的actualValueNow方法。

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


