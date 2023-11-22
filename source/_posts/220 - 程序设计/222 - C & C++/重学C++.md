---
title:  重学C++
date: 2023-11-21 16:27:41
categories:
  - C & C++
tags:
  - 程序设计
  - C++
published: false
---

## C++程序存储区域划分
存储区域划分为：
1. 栈区
2. 堆区
3. 全局未初始化变量
4. 全局初始化变量
5. 代码区
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/1.png)
在内存中分配区域时，按上图中的箭头方向，申请的变量位于栈区时，从高地址往低地址分配，堆区相反。

- **栈区和堆区对比**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231117212913.png)

- **全局静态存储区和常量存储区对比**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231117213238.png)

## C++中的智能指针
### auto_ptr
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231117213609.png)
由new expression 获得对象，在auto_ptr 对象销毁时，他所管理的对象也会自动被 delete 掉。

**所有权转移:** 不小心把它传递给另外的智能指针，原来的指针就不再拥有这个对象了在拷贝/赋值过程中，会直接剥夺指针对原对象对内存的控制权，转交给新对象，然后再将原对象指针置为空。
> auto_ptr 在C++17 以后已经作废，不建议在真实项目中使用

```cpp
#include <iostream>
#include <memory>
using namespace std;

void auto_ptr_test() {
    auto_ptr<int> pl(new int(10));
    cout << *pl << endl; // 10

    auto_ptr<string> languages[5] = {
            auto_ptr<string>(new string("C")),
            auto_ptr<string>(new string("Java")),
            auto_ptr<string>(new string("C++")),
            auto_ptr<string>(new string("Python")),
            auto_ptr<string>(new string("Rust")),
    };

//    for (const auto & language : languages) {
//        cout << *language << endl;
//    }

    // languages[2] 将所有权转让给pC，此时languages[2]不再引用该字符串，从而变成空指针
    auto_ptr<string> pC = languages[2];
    cout << *pC << endl;  // C++

    cout << "is: " << *languages[2] << endl; // is:
}

int main() {
    auto_ptr_test();
    return 0;
}
```


### unique_ptr
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231117205001.png)
`unique_ptr`是专属所有权，所以`unique_ptr`管理的内存，只能被一个对象持有不支持复制和赋值。
**移动语义**：`unique _ptr` 禁止了拷贝语义，但有时我们也需要能够转移所有权，于是提供了移动语义，即可以使用`std::move()`进行控制所有权的转移。

```cpp
void unique_ptr_test() {  
    auto w = std::make_unique<int>(10);  
    cout << *(w.get()) << endl;   // 10
//    auto w2 = w; // 编译报错，不可以复制  
  
    // unique_ptr 只支持移动语义  
    auto w2 = std::move(w);  
    cout << (w.get() != nullptr ? *w.get() : -1) << endl;  // -1
    cout << (w2.get() != nullptr ? *w2.get() : -1) << endl; // 10
}
```

### shared_ptr 和 weak_ptr
- **shared_ptr**

shared_ptr通过一个引用计数共享一个对象，shared_ptr为了解决auto_ptr在对象所有权上的局限性，通过使用引用计数的机制，提供了可以共享所有权的智能指针。
当引用计数为0时，该对象没有被使用，可以进行析构。

```cpp
void shared_ptr_test() {
    // shared_ptr 代表共享所有权，多个 shared_ptr 可以共享同一块内存
    auto wA = std::make_shared<int>(20);
    {
        auto wA2 = wA;
        cout << (wA2.get() != nullptr ? *wA2.get() : -1) << endl;  // 20
        cout << (wA.get() != nullptr ? *wA.get() : -1) << endl;  // 20
        cout << wA2.use_count() << endl;  // 2
        cout << wA.use_count() << endl;  // 2
    }

    cout << wA.use_count() << endl;  // 1

    auto wAA = std::make_shared<int>(30);
    auto wAA2 = std::move(wAA);
    cout << (wAA.get() != nullptr ? *wAA.get() : -1) << endl;  // -1
    cout << (wAA2.get() != nullptr ? *wAA2.get() : -1) << endl;  // 30
    cout << wAA.use_count() << endl;  // 0
    cout << wAA2.use_count() << endl;  // 1
    // 将wAA 对象 move 给 wAA2，意味着 wAA 放弃了对内存的所有权和管理，此时 wAA2 对象等于 nullptr
    // 而wAA2 获得了对象所有权，但因为wAA已不再持有对象，因此wAA2的引用计数为1
}
```

- **weak_ptr**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231117211444.png)
循环引用导致引用计数永远不为0，会导致堆里的内存无法正常回收，造成内存泄漏。

weak_ptr 与 shared_ptr 共同工作，用一种观察者模式工作，协助 shared_ptr 工作。
weak_ptr 只对 shared_ptr进行引用，不改变其引用计数，当被观察的shared_ptr失效后，相应的 weak_ptr 也相应失效。
```cpp
#include <iostream>  
#include <memory>  
using namespace std;

struct B;
struct A {
    shared_ptr<B> pb;
    ~A(){
        cout << "~A()" << endl;
    }
};

struct B {
    shared_ptr<A> pa;
    ~B(){
        cout << "~B()" << endl;
    }
};

void Test() {
    shared_ptr<A> tA(new A());
    shared_ptr<B> tB(new B());

    cout << tA.use_count() << endl;  // 1
    cout << tB.use_count() << endl; // 1

    tA->pb = tB;
    tB->pa = tA;
    cout << tA.use_count() << endl; // 2
    cout << tB.use_count() << endl; // 2
}

int main() {
	Test();
    return 0;
}
```
输出：
```
1
1
2
2
```
以上出现循环引用，最后没有输出析构函数的操作。


```cpp
#include <iostream>  
#include <memory>  
using namespace std;

struct BW;
struct AW {
    shared_ptr<BW> pb;
    ~AW(){
        cout << "~AW()" << endl;
    }
};

struct BW {
    weak_ptr<AW> pa;
    ~BW(){
        cout << "~BW()" << endl;
    }
};

void Test2() {
    shared_ptr<AW> tA(new AW());
    shared_ptr<BW> tB(new BW());

    cout << tA.use_count() << endl;
    cout << tB.use_count() << endl;

    tA->pb = tB;
    tB->pa = tA;
    cout << tA.use_count() << endl;
    cout << tB.use_count() << endl;
}

int main() {
	Test2();
    return 0;
}
```
输出：
```
1
1
1
2
~AW()
~BW()
```
通过使用 weak_ptr ，程序正常执行了 析构函数。


