---
title: C++编程思想笔记
date: 2019-03-15 17:39:38
tags:
---

1.改变已存在的基类函数的行为，称为重载这个函数？

2.面向对象编程的宗旨

- 系统中有哪些对象？
- 它们之间的接口是什么？

3.函数声明时参数标识符是可选的。函数定义时则要求要有标识符。(C++中允许不命名参数为了给程序员提供在参数列表中保留位置)

4.关于#include ：永尖括号来指定文件时，预处理器是以特定的方式来寻找文件，一般是环境中或编译器命令行指定的某种寻找路径。用双引号时，预处理器以“由实现定义的方式”来寻找文件。它通常是从当前目录开始寻找，如果没有找到，那么include 命令按与尖括号同样的方式开始寻找。

5.名字空间的作用：当程序达到一定规模后，通常分成许多块，每一块由不同的人或小组来构造和连接。不同模块中碰巧用到了相同的函数名和标识符会导致不必要的冲突，标准的C++中用namespace的机制防止这种冲突。

6.string 具有动态特性，不必担心string的内存分配；只管添加新的内容进去就行了，string会自动扩展以保存新的输入。

7.C++函数原型必须指明函数的返回值类型

8.static变量的两层意义

- 如果想使局部变量的值在程序整个生命期里仍然存在，我们可以定义函数的局部变量为static
- static的第二层意思是“在某个作用域外不可访问”，函数名是局部于文件的，我们说它具有文件作用域

9.extern关键字的意义

- 它告诉编译器存在着一个变量和函数，即使编译器在当前编译的文件中没有看到它。这个变量或函数可能在另一个文件中或者当前文件后面定义。

10.C++中的显示转换

| static_cast      | 用于“良性”和“适度良性”转换，包括不用强制转换                 |
| ---------------- | ------------------------------------------------------------ |
| const_cast       | 对"const"和/或"volatile"进行转换(从const转换为非const或从volatile转换为非volatile) |
| reinterpret_cast | 重解释转换                                                   |
| Dynamic_cast     | 用于类型安全的向下转换                                       |

11.typedef 给struct定义别名

```
typedef struct SelfReferential {
 	int i;
    SelfReferential* sr;
}SelfReferential;

//使用
SelfReferential sr1, sr2;
```

12.使用预处理调试标记

```
#define DEBUG
#ifdef DEBUG
/*调试的代码*/
#endif DEBUG
```

13.在写调试代码时，编写包含变量名后跟变量名是很无聊的，可以用字符串化运算符简化

```
#define P(A) cout << #A << ": " << (A) << endl;
```

```
int main() {
    int a = 1, b = 2, c = 3;
    P(a); P(b); P(c);
    P(a + b);
    P((c - b) / b);
}
```

```
[root@localhost CPPDemo]# ./cast1 
a: 1
b: 2
c: 3
a + b: 3
(c - b) / b: 0
```

14.一旦定义了一个函数指针，在使用前必须给它赋一个函数的地址

```
void func() {
    cout << "func() called ..." << endl;
}

void func2() {
    cout << "func2() called .." << endl;
}

int main() {
    void (*fp)(); //定义一个函数指针
    fp = func; //初始化它
    (*fp)(); //调用它指向的函数
    void (*fp2)() = func2; //定义并初始化一个函数指针
    (*fp2)();
}
```

```
[root@localhost CPPDemo]# ./fun_ptr 
func() called ...
func2() called ..
```

15.妙用函数指针数组

```
#include <iostream>
using namespace std;

#define DF(N) void N() {\ 
    cout << "function " #N " called ... " << endl; }

DF(a); DF(b); DF(c); DF(d); DF(e); DF(f); DF(g);

void (*func_table[])() = {a, b, c, d, e, f, g};

int main() {
    while(1) {
        cout << "press a key from 'a' to 'g'"
            "or q to quit" << endl;
        char c, cr;
        cin.get(c); cin.get(cr);
        if (c == 'q')
            break;
        if (c < 'a' || c > 'g')
            continue;
        (*func_table[c - 'a'])();

    }
}

[root@localhost CPPDemo]# ./fun_ptr_table 
press a key from 'a' to 'g'or q to quit
b
function b called ... 
press a key from 'a' to 'g'or q to quit
c
function c called ... 
press a key from 'a' to 'g'or q to quit
q
```

16.C++允许将任何类型的指针赋值给void *，但不允许指针赋给任何其他类型的指针。

17.C++中的访问控制

| public    | 在其后声明的所有成员可以被所有的人访问                       |
| --------- | ------------------------------------------------------------ |
| private   | private关键字意味着，除了该类型的创建者和类的内部成员函数之外，任何人都不能访问 |
| protected | 继承的结构可以访问protected成员，但不能访问private成员       |

18.访问控制为类的创建者提供了很有价值的控制。类的客户程序员可以清楚的看到，什么可以用，什么应该忽略。更重要的是，它保证了使用类的客户程序员不会依赖类的实现细节，这样类库的开发者更改某些实现客户程序员不会因此受到影响。

19.构造函数执行原理

```
class X {
    int i;
public:
	X();
}

void f() {
    X a;
    // ...
}
```

当程序执行到a的序列点执行的点时，构造函数自动被调用，编译器悄悄的在a的定义点处拆入了一个X::X()的调用。就像其他成员函数被调用一样，传递到构造函数的第一个(秘密)参数是this指针，也就是调用这一个函数对象的内存地址，不过，对构造函数来说，this指针指向一个没有被初始化的内存块，构造函数的作用就是正确初始化该内存块

20.当对象超出它的作用域的时候，编译器将自动调用析构函数。

21.C++中禁止用返回值重载函数

22.在C++中，struct和class唯一的不同之处在于，struct默认为public，而class默认为private。自然的，也可以让struct有构造啊函数和析构函数。另外，一个union（联合）也可以带有构造函数、析构函数、成员函数甚至访问控制。

23.只有参数列表的后部参数才是可默认的；一旦在一个函数调用中开始使用默认参数，那么这个参数后面所有参数都必须是默认的。

24.占位符参数:函数声明时，参数可以没有标识符

```
void f(int x, int, float flt);
```

如果开始用了一个函数参数，而后来发现不需要用它，可以将它去掉而不会产生警告错误，而且不需要改动那些调用该函数以前版本的程序代码。

25.不能把默认参数作为一个标志去决定执行函数的哪一块，这是基本原则。在这种情况下，只要能够，就应该把函数分解成两个或多个重载的函数。

26.关于const和指针

u是一个指针，它指向一个const int，它所指的内容是不能被改变的

```
const int* u;
```

w是一个指针，这个指针是指向int的const指针，指针本身不可改变

```
int d = 1;
int* const w = &d;
```

指针的对象都不能改变

```
int d = 1;
const int* const x = &d
```

