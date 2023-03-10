---
title: C++函数重载
date: 2022/3/25
categories:
- C语言
---

@[TOC](目录)

---

重载是C++新增的机制，将语义和功能相似的函数用同一个名字表示，提高函数的通用性。

# 重载

特征：
（1）相同范围
（2）函数名相同
（3）参数不同
（4）virtual可有可无
全局函数和成员函数即使名称和参数相同，两者也是互不干扰。
在同一范围内，各种参数类型的`MyPrint`函数都可以直接调用，只要明确实参的类型即可。
```cpp
#include <iostream>
using namespace std;

void MyPrint(int num)
{
  cout << "int: " << num << endl;
}

void MyPrint(float num)
{
  cout << "float: " << num << endl;
}

class Test
{
  public:
         void MyPrint(int num) {cout << "class int: " << num << endl;}
         void MyPrint(float num) {cout << "class float: " << num << endl;}
         void MyPrint(char num) {cout << "class char: " << num << endl;}
};

int main(void)
{
  MyPrint(1); // int: 1
  MyPrint(1.1f); // float: 1 浮点型必须要显式类型，否则编译器不知道该转换为int还是float。
  Test test1;
  test1.MyPrint(2); // class int: 2
  test1.MyPrint(2.1f); // class float: 2.1
  test1.MyPrint("helloc"); // class char: helloc
  return 0;
}
```

---
---

# 覆盖

特征：
（1）不同范围（指派生类和基类）
（2）函数名相同
（3）参数相同
（4）基类函数的virtual关键词必须有
基类`h(int a)`被派生类的`h(int a)`所覆盖
```cpp
#include <iostream>
using namespace std;

class Base
{
  public:
  virtual void h(int a) {cout << "Base h(int): " << a << endl;}
}

class Derived : public Base
{
  public:
    void h(int a) {cout << "h(int): " << a << endl;}
}

int main(void)
{
  Derived dr;
  Base *p = &dr;
  p->h(2); // h(int): 2
  return 0;
}
```

---
---

# 隐藏

隐藏是指派生类的函数隐藏了基类的同名函数。
特征：
（1）不同范围（派生类和基类）
（2）函数名相同
（3）若参数不同，则基类函数被隐藏；若参数相同，且基类没有`virtual`关键词，则隐藏。（有`virtual`被覆盖）

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    void f(int a) {cout << "Base f(int): " << a << endl;} //隐藏
    void g(int a) {cout << "Base g(int): " << a << endl;} //隐藏
virtual void h(int a) {cout << "Base h(int): " << a << endl;} //覆盖
};

class Derived : public Base
{
public:
    void f(float a) {cout << "f(int): " << a << endl;}
    void g(float a) {cout << "g(int): " << a << endl;}
    void h(int a) {cout << "h(int): " << a << endl;}
};

int main(void)
{
    Derived d;
    Base *p = &d;

    p->f(1); // Base f(int): 1
    d.f(2); // f(int): 2

    p->g(3); // Base g(int): 3
    d.g(4); // g(int): 4

    p->h(5); // h(int): 5
    d.h(6); // h(int): 6
    return 0;
}
```

---
---

# 重载运算符

在C++中可以用`operator`加上运算符来替代函数名，表示函数。
下例程序完成`+`和`-`运算符的重载，需要注意的是，`+`函数定义了2个形参，必须要使用`friend`关键词，将成员函数改为友员函数，友员函数没有`this`指针。全局函数和友员函数相同。
成员函数默认第一个形参省略，用`this`指针代替。

```cpp
#include <iostream>
using namespace std;

class Symbol
{
public:
    friend Symbol operator +(Symbol& a, Symbol& b)
    {
        Symbol s;
        s.Length = a.Length + b.Length;
        s.Width = a.Width + b.Width;
        s.Height = a.Height + b.Height;
        return s;
    }
    Symbol operator -(Symbol& a)
    {
        Symbol s;
        s.Length = this->Length - a.Length;
        s.Width = this->Width - a.Width;
        s.Height = this->Height - a.Height;
        return s;
    }

    int Length;
    int Width;
    int Height;

};

int main(void)
{
    Symbol a;
    Symbol b;
    Symbol s;

    a.Length = 1;
    a.Width = 2;
    a.Height = 3;

    b.Length = 4;
    b.Width = 5;
    b.Height = 6;

    s = a + b;

    cout << "s.Length: " <<s.Length << "\ns.Width: " << s.Width \
    << "\ns.Height: " << s.Height << endl;
    // s.Length: 5 
    // s.Width:7
    // s.Height:9
    
    s = b - a;

    cout << "\ns.Length: " <<s.Length << "\ns.Width: " << s.Width \
    << "\ns.Height: " << s.Height << endl;
    // s.Length: 3
    // s.Width:3
    // s.Height:3
    
    return 0;
}
```

从语法规则上来说，运算符既可以定义为全局函数，又可以定义为成员函数，但因为一些运算符的常用性，有以下建议：
（1）一元运算符：建议重载为成员函数；
（2）`=, (), [], ->` ：只能重载为成员函数；
（3）`+=, -=, /=, *=, &=, |=, ~=, %=, >>=, <<= `：建议重载为成员函数。

为了防止错误和混乱，有一些运算符是不允许被重载的：
（1）数据类型（`int, float, double, char`等运算符）；
（2）成员访问运算符`.`；
（4）C++运算符集合没有的符号，比如`@, $`，预处理符`#`；

