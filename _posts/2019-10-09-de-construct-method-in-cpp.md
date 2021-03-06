---
title: "构造函数和析构函数指南"
---



构造和析构是面对对象程序设计中极其重要的两个概念，虽然其原理比较简单，但是知识点很琐碎，而且两者的使用基本会贯穿整个程序。这篇文章希望能够尽量完整详细的梳理c++中构造函数和析构函数的使用。



- [简介](#简介)

- [语法要求](#语法要求)

- [构造函数的几种类型](#构造函数的几种类型)
  - [默认构造函数](#默认构造函数)
  - [拷贝构造函数](#拷贝构造函数)
  - [移动构造函数](#移动构造函数)
- [析构函数](#析构函数)
- [object-based](#object-based)
  - [成员的构造与析构](#成员的构造与析构)
  - [委派构造函数](#委派构造函数)
- [object-oriented](#object-oriented)
  - [继承关系中的构造函数](#继承关系中的构造函数)
  - [继承关系中的析构函数](#继承关系中的析构函数)
  - [虚函数](#虚函数)
- [RAII](#RAII)
- [其他](#其他)
  - [不需要提供参数的构造函数](#不需要提供参数的构造函数)
  - [default关键字](#default关键字)
  - [=delete](#=delete)
  - [什么时候需要自定义复制/移动构造函数](#什么时候需要自定义复制/移动构造函数)
  - [继承与移动构造函数](#继承与移动构造函数)
  - [构造函数、析构函数与异常](#构造函数、析构函数与异常)





## 简介

构造函数，顾名思义，是用来创建对象或者初始化对象。

析构函数，对应地，用来销毁对象或者回收资源。

更详细地，以资源为度量，构造函数就是申请资源，包括对象所管理的额外资源；析构函数就是释放资源，同时释放被管理的资源。这是构造和析构的根本作用。



可以把构造函数和析构函数看作是特殊的成员函数，其特殊之处非常多，会在后面一一记录。



> "ctor" *是构造函数*(constructor)典型的缩写。



## 语法要求

首先从c++语法角度来说明构造函数和析构函数的特殊之处。



- **函数签名**
  - 名称。构造函数的名称和类名相同，析构函数的名称是"~"后跟类名。
  - 返回值。构造函数的析构函数都有没返回值。
  - 参数。构造函数可接受除了此类实例外的任意参数，析构函数不能带有参数。



其他还有：

- 不能是const 类型。因此在创建一个`const`类型的对象时，只有在构造完成后，才获得`const`属性。
- 构造函数不能是虚函数
- 析构函数没有参数，所以不接受重载
- static成员不与实例对象关联，所以不能在构造函数中初始化。同样，析构函数不会破坏任何static变量。



## 构造函数的几种类型

构造函数可以接收参数，而且根据接收参数的不同，构造函数会有不同的用途和叫法：

- default constructor，默认构造函数，不需要提供参数的构造函数[注](#不需要提供参数的构造函数)
- conversion constructor，转换构造函数，只有一个参数，且此参数不是本类的引用。常常希望避免隐式转换，可使用**explicit**关键字
- copy constructor，拷贝构造函数，只有一个参数需要提供值，且此参数是本类的一个左值引用
- move constructor，移动构造函数，只有一个参数需要提供值，且此参数是本类的一个右值引用
- 普通构造函数



### 默认构造函数

当没有定义任何构造函数的时候，编译器会创造一个构造函数，叫做**合成的默认构造函数**，它按照如下规则初始化数据成员：

- 如果存在类内初始值，用它来初始化
- 否则，默认初始化该成员。默认值是多少，由变量类型决定



如果类内包含一个其他类类型的成员**，**而且该成员的类型没有默认构造函数，则无法提供合成的默认构造函数。必须**显式声明默认构造函数**。



如果已经定义了一个构造函数，不管这个构造函数是不是默认构造函数，编译器都不再提供合成的默认构造函数。不过在c++11中，可以使用default关键字强制编译器自动生成合成的默认构造函数。



### 拷贝构造函数

拷贝构造函数的严格说法如下：

> 对于一个类X，如果一个构造函数的第一个参数是下列之一：
>
> - X&
> - const X&
> - volatile X&
> - const volatile X&
>
> 且没有其他参数或其它参数都有默认值，那么这个函数就是拷贝构造函数。



而且类中可以存在超过一个的拷贝构造函数。



如果一个类中没有定义拷贝构造函数，那么编译器会自动产生一个**默认的拷贝构造函数**。

**默认拷贝构造函数**执行浅拷贝，其行为如下：
拷贝构造函数对类中每一个非static数据成员执行成员拷贝(memberwise Copy)的动作。

- 如果数据成员为某一个类的实例，那么调用此类的拷贝构造函数。
- 如果数据成员是一个数组，对数组的每一个元素按位拷贝。
- 如果数据成员是一个内置类型，如int，double，那么调用系统内建的赋值运算符对其进行赋值。



触发拷贝构造函数的场景：

1. 初始化。`T a = b;`  ` T a(b);` b的类型也是T。
2. 函数参数传递。`f(a);`  a的类型是T，且f的签名为`void f(T t)`。
3. 函数的返回值。`T f()` ，T类型没有移动构造函数。



> 拷贝构造函数的第一个参数必须是引用类型，否则当这个类需要使用拷贝来传递值的时候（比如函数形参和实参）会陷入无限循环。





### 移动构造函数

从c++11才有。

跟拷贝构造函数一样的定义：其首个形参是 T&&、const T&&、volatile T&& 或 const volatile T&&，且无其他形参，或剩余形参均有默认值。



当一个对象实例被一个同类型的右值初始化（直接初始化或者复制初始化），移动构造函数将会被调用，这些场景有：

1. 初始化。`T a = std::move(b);`  ` T a(std::move(b));` b的类型也是T。
2. 函数参数传递。`f(std::move(a));`  a的类型是T，且f的签名为`void f(T t)`。
3. 函数的返回值。`T f()` ，T类型必须有移动构造函数。



类中也可以存在多个版本的移动构造函数。



当我们定义了自己的拷贝构造函数或者析构函数之后，编译器不会合成默认的移动构造函数。

因为，如果我们定义了这些操作往往表示类内含有指针成员需要动态分配内存，如果需要为类定义移动操作，那么应该确保移动后源对象是安全的，但是默认的移动构造函数不会帮我们把指针成员置空，移后源不是可析构的安全状态，如果这样，当离开移动构造后，源对象被析构，对象内的指针成员空间被回收，转移之后对象内的指针成员出现悬垂现象，程序将引起致命的错误。所以当我们定义了自己的拷贝操作和析构函数时，编译器是不会帮我们合成默认移动构造函数的。



## 析构函数

是构造函数的互补，当对象超出作用域或动态分配的对象被删除时，将自动应用析构函数。析构函数可用于释放对象时构造或在对象的生命期中所获取的资源。



析构函数的运行：

当对象引用或指针越界的时候不会执行析构函数，只有在删除指向动态分配对象的指针或实际对象超出作用域时才会调用析构函数。



合成析构函数：

编译器总是会合成一个析构函数，合成析构函数按对象创建时的逆序撤销每个非static成员。要注意的是，合成的析构函数不会删除指针成员所指向的对象。



一个类如果需要管理一些类外资源（例如，动态分配的资源），那么这个类需要通过自定义析构函数来释放对象多分配的资源，同时也应该定义拷贝控制成员。所以可作如下推断：**如果一个类需要自定义析构函数，几乎可以肯定它也需要自定义拷贝赋值运算符和拷贝构造函数；**如果一个类需要一个拷贝构造函数，几乎可以肯定它也需要一个拷贝赋值运算符，反之亦然，但是不一定需要析构函数。



## object-based

基于对象是一种Abstract DataType，只是将对象抽象成一种数据类型，并不会通过继承产生新的数据类型。所以基于对象只用到了“封装、继承、多态”中的封装。这样理解的话，基于对象类似于c语言中的结构体struct。



### 成员的构造与析构

类中成员变量的初始化有两种方式：

1. 构造函数的**初始化列表**
2. 构造函数体内赋值初始化



初始化和赋值对内置类型的成员没有什么大的区别。但对非内置类型成员变量，为了避免两次构造，**推荐使用初始化列表**。而且有的时候必须用带有初始化列表的构造函数：

1. **成员变量没有默认构造函数**。若没有提供显示初始化式，则编译器隐式使用成员变量的默认构造函数，若类没有默认构造函数，则编译器尝试使用默认构造函数将会失败。
2. **有const成员变量或引用类型的成员变量**。因为const对象或引用类型只能初始化，不能对他们赋值。





> 初始化列表的成员变量初始化顺序，取决于它们在类中出现的顺序，与初始化列表中的顺序无关。而且初始化列表的执行顺序在构造函数体之前。



### 委派构造函数

委派构造函数是C++11标准的新特性，其使用类中的一个构造函数通过初始化列表方式来调用同一个类中的另一个构造函数。一个典型用法就是将所有成员的初始化集中在某一个构造函数中，然后其他构造函数通过委派构造函数的方式调用这个构造函数。

但要注意不能形成死循环。



## object-oriented



### 继承关系中的构造函数

**派生类构造函数调用顺序**如下：

1. **基类构造函数**。如果有多个基类，则构造函数的调用顺序是基类在类派生表中出现的顺序。
2. **若派生类中包含对象成员，还要进行对象成员初始化**。如果有多个成员类对象则构造函数的调用顺序是对象在类中被声明的顺序。
3. [派生类构造函数](#成员的构造与析构)。





如果父类没有提供默认构造函数，那么在子类构造函数中就必须显式提供父类的初始化。



当派生类定义自己的拷贝构造函数时，也要注意使用初始化列表的方式初始化基类。



类不能继承默认、拷贝、移动构造函数。





### 继承关系中的析构函数

**析构函数的析构顺序**正好和构造函数[相反](#继承关系中的构造函数)。

1. 执行派生类的析构函数体。

2. 销毁数据成员，与创建的顺序相反。

3. 如果有父类，调用父类的析构函数。





与构造函数和赋值运算符不同的是，析构函数只负责销毁派生类自己分配的资源。基类的析构函数是隐式调用的。



大多数基类都会定义一个虚析构函数，因此默认情况下，基类通常不含有合成的移动操作，而且在他的派生类中也没有合成的移动操作。如果确实需要执行移动操作应该首先在基类中进行定义。



### 虚函数

任何构造函数之外的非静态函数都可以是虚函数（virtual）。



如果构造函数或析构函数调用了某个虚函数，则应该执行与构造函数或析构函数所属类型相对应的虚函数版本（也就是说没有动态绑定）。因为构造函数负责建立虚函数表，析构函数负责撤销虚函数表，在这两个函数中调用虚函数时，虚函数表都是不能保证完整的。但是最好不要这样做。



基类通常应该定义一个虚析构函数，即使该函数不执行任何实际操作。这样做的目的是为了当基类指针指向派生类对象时，可以通过基类指针安全删除派生类对象，否则可能造成内存泄漏。



## RAII

全称是“Resource Acquire Is Initial”，是一种 C++ 编程技术，它将必须在使用前请求的资源（分配的堆内存、执行线程、打开的套接字、打开的文件、锁定的互斥体、磁盘空间、数据库连接等——任何存在受限供给中的事物）的生命周期绑定与一个对象的生存期相绑定。



通俗点解释就是：**在类的构造函数中分配资源，在析构函数中释放资源**。这样，当一个对象创建的时候，构造函数会自动地被调用；而当这个对象被释放的时候，析构函数也会被自动调用。于是乎，一个对象的生命期结束后将会不再占用资源，资源的使用是安全可靠的。



RAII 可总结如下:

- 将每个资源封装入一个类，其中
- 构造函数请求资源，并建立所有class invariants，或在它无法完成时抛出异常，
- 析构函数释放资源并决不抛出异常；
- 始终通过 RAII 类的实例使用资源
  - 要么自身拥有自动存储期或临时生存期（例如，栈上的变量）
  - 要么具有与自动或临时对象的生存期绑定的生存期（例如，智能指针）



总之多用智能指针，少写delete。





## 其他



### 不需要提供参数的构造函数

不需要提供参数有两种可能：

- 这个构造函数没有参数
- 这个构造函数的所有参数都有默认值

这两种形式的构造函数都可以作为默认构造函数，但是不能同时出现在一个类中，否则当使用默认构造函数的时候会报错——“此类有多个默认构造函数”。

也就是说默认构造函数只能有一个。



### default关键字

c++11才有

Defaulted 函数特性仅适用于类的六个特殊成员函数，且该特殊成员函数没有默认参数。

1.默认构造函数
2.默认析构函数
3.拷贝构造函数
4.拷贝赋值函数
5.移动构造函数
6.移动拷贝函数

C++规定，一旦程序员实现了这些函数的自定义版本，则编译器不会再自动生产默认版本。注意只是不自动生成默认版本，当然还是可手动生成默认版本的。当我们自己定义了待参数的构造函数时，我们最好是声明不带参数的版本以完成无参的变量初始化，此时编译是不会再自动提供默认的无参版本了。我们可以通过使用关键字default来控制默认构造函数的生成，显式地指示编译器生成该函数的默认版本。



### =delete

c++11才有。

可在想要 “禁止使用” 的特殊成员函数声明后加 “= delete”（当然也可以声明为私有函数或者保护函数）

不同于default 只能使用在类的特殊函数上，**可以对任何函数使用`=delete`**。但是对析构函数使用delete，编译器将不允许定义该类型的变量或临时对象，也就会导致无法销毁此类型的实例。

一个用途就是通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数（deleted function）来阻止拷贝。例如**`iostream`类就阻止拷贝**

`=delete`必须出现在函数第一次声明地时候。

在某些情况下，编译器会将这些合法的成员函数定义为删除函数，本质上，这些规则的含义就是，如果一个类有数据成员不能默认构造、拷贝、复制、销毁，则对应的成员函数将被定义为删除的：

1. 如果类的某个成员的析构函数不可访问（private）或者是删除的（delete），则类的合成析构函数被定义为删除的。
2. 如果类的某个成员的拷贝构造函数是删除的或不可访问的，则类的合成拷贝构造函数被定义为删除的；同样，如果类的某个成员的析构函数是删除的或不可访问的，则类合成的拷贝构造函数也被定义为删除的。
3. 如果类的某个成员的拷贝赋值运算符是删除的或者不可访问的，或者类有一个const的引用成员，则类的合成拷贝赋值运算符被定义为删除的。
4. 如果类的某个成员的析构函数是删除的或不可访问的，或是类有一个引用成员，但它没有类内初始化器，或者类有一个const成员，他没有类内初始化器且其类型未显式定义默认构造函数，则该类的默认构造函数被定义为删除的。





### 什么时候需要自定义复制/移动构造函数

只包含类类型或内置类型数据成员、不含指针的类一般可以使用合成操作，复制、赋值或撤销这样的成员不需要特殊控制。

换言之，如果数据中有指针，为了避免浅复制，就需要定义自己的复制构造函数、赋值操作符和析构函数了。



### 继承与移动构造函数

```c++
Base(Base const & rhs); // 拷贝构造函数
Base(Base&& rhs); //移动构造函数

Derived(Derived const & rhs) : Base(rhs) {}; // 正确
Derived(Derived&& rhs)
    :Base(rhs)      // 错误，rhs是左值，会调用Base的拷贝构造函数
{};

Derived(Derived&& rhs):Base(std::move(rhs)) {}; // 正确
```



### 构造函数、析构函数与异常

构造函数：

1. 构造函数中抛出异常将导致对象的析构函数不被执行。C++仅能 delete 被完全构造的对象（fully constructed objects）
2. 因为析构函数不能被调用，所以可能会造成内存泄露或系统资源未被释放。
3. 当对象发生部分构造时，已经构造完毕的子对象（非动态分配）将会逆序地被析构。



析构函数：

1. C++标准指明**析构函数不能、也不应该抛出异常。**
2. 一个必要的解决方法就是把异常完全封装在析构函数内部，决不让异常抛出函数之外。



所以结论如下：

- 构造函数中尽量不要抛出异常。但也可以抛出异常，但必须保证在构造函数抛出异常之前，把系统资源释放掉，防止内存泄露。

- 不要在析构函数中抛出异常。

