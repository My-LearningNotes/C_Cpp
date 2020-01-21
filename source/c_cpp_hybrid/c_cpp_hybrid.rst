C和C++混合编程
==============

在使用C++的项目源码中，经常会看到类似下面的代码:

.. code-block:: c++

    #ifdef __cplusplus
    extern "C" {
    #endif 
       
    /* ... */
       
    #ifdef __cplusplus
    }
    #endif


它到底有什么用？下面就从以下几个方面来介绍它:

-  ``#ifdef/#endif``, ``__cplusplus``\ 及发散

-  ``extern "C"``

    -  extern关键字

    -  ``"C"``

    -  小结\ ``extern "C"``

-  C和C++相互调用

    -  C++的编译和链接

    -  C的编译和链接

    -  C++中调用C的代码

    -  C中调用C++的代码

-  C和C++混合调用特别之处 - 函数指针


#ifdef/#endif, \__cplusplus及发散
---------------------------------

``#ifdef/#endif``, ``#ifndef/#endif``\ 用于条件编译。

.. code-block:: c++

    #ifdef __cplusplus
    extern "C"{
    #endif

上面的代码表示如果定义了宏\ ``__cplusplus``\ ，就执行\ ``#ifdef/#endif``\ 之间的语句，否则就不执行。

这里为什么需要条件编译呢？

因为C语言中不支持\ ``extern "C"``\ 声明，如果明白了\ ``extern "C"``\ 的作用就知道在C中也没有必要这样做，在\ *.c*\ 文件中包含了\ ``extern "C"``\ 时会出现编译时错误。

既然说到了条件编译，它还有一个重要应用 --- *避免头文件的重复包含*\ 。


``extern "C"``
--------------

首先从字面上分析\ ``extern "C"``\ ，它由两部分组成: ``extern``\ 关键字和\ ``"C"``\ ，下面就从这两个方面来解读\ ``extern "C"``\ 的含义。


``extern``\ 关键字
~~~~~~~~~~~~~~~~~~

在一个项目中，必须保证函数，变量，枚举等在所有的源文件中是唯一的(不能重复定义)，除非将其定义为局部的。

首先来看一个例子:

.. code-block:: c

    // file1.c
    int x = 1;
    int f() {do something here}

    // file2.c
    extern int x;
    int f();
    void g(){x = f();}

在\ *file2.c*\ 中\ ``g()``\ 使用的\ ``x``\ 和\ ``f()``\ 是定义在\ *file1.c*\ 中的。

``extern``\ 关键字表明\ *file2.c*\ 中的\ ``x``\ ，只是一个变量的声明，其并不是在定义变量\ ``x``\ ，并未为\ ``x``\ 分配内存空间。
变量\ ``x``\ 在所有模块中作为一个全局变量只能被定义一次，否则编译时会报变量重复定义的错误。
一个变量只能定义一次，但是可以声明多次，且声明时必须保证类型一致。

例如:

.. code-block:: c

    // file1.c
    int x = 1;
    int b = 1;
    extern c;

    // file2.c
    int x;  // x equals to default of int type 0
    int f();
    extern double b;
    extern int c;

在这段代码中有三个错误:

- ``x``\ 被定义了两次

- ``b``\ 两次被声明为不同的类型

- ``c``\ 被声明了两次，但却没有定义

回到\ ``extern``\ 关键字，\ ``extern``\ 是C/C++语言中表明\ **函数**\ 和\ **全局变量**\ 作用范围(可见性)的关键字，该关键字告诉编译器，其声明的函数和变量可以在本模块或其它模块中使用。
通常，在模块的头文件中对本模块提供给其它模块引用的函数和全局变量以关键字\ ``extern``\ 声明。
例如，如果模块B欲引用模块A中定义的全局变量和函数时，只需包含模块A的头文件即可。
这样，模块B中调用模块A中的函数时，在编译阶段，模块B虽然找不到该函数，但是并不会报错；它会在链接阶段从模块A编译生成的目标代码中查找此函数。

与\ ``extern``\ 对应的关键字是\ ``static``\ ，被它修饰的全局变量和函数只能在本模块中使用。

因此，一个函数或变量只可能在本模块中使用时，其不可能被\ ``extern "C"``\ 修饰。


``"C"``
~~~~~~~

一个C++程序中可能包含其它语言编写的部分代码，类似的，用C++编写的代码片段也可能被使用在其它语言编写的代码中。
不同语言编写的代码相互调用是困难的，甚至是同一种语言编写但不同的编译器编译的代码。
例如，不同语言和同种语言的不同实现，可能会在注册变量保持参数和参数在栈上的布局等方面都不一样。

为了使它们遵守统一的规则，可以使用\ ``extern``\ 指定一个编译和链接规约。
例如，声明C和C++标准库函数\ ``strcpy()``\ ，并指定它应该根据C的编译和链接规约来链接:

.. code-block:: c

    extern "C" char* strcpy(char*, const char*);

注意它与下面的声明的不同之处:

.. code-block:: c

    extern char* strcpy(char*, const char*);

下面的这个声明仅表示在链接的时候调用\ ``strcpy()``\ 。

``extern "C"``\ 指令非常有用，因为C和C++的近亲关系。

**注意，extern "C"指令中的"C"，表示的是一种编译和链接规约，而不是一种语言。**

还有要说明的是，\ ``extern "C"``\ 指令仅指定编译和链接规约，但不影响语义。
例如，在函数声明中指定了\ ``extern "C"``\ ，仍然要遵守C++的类型检测，参数转换规则。

为了声明一个变量而不是定义一个变量，需要在声明时指定\ ``extern``\ 关键字，但是当又加上了\ ``"C"``\ ，它不会改变语义，但是会改变它的编译和链接方式。
如果有很多对象都要加上\ ``extern "C"``\ ，可以将它们放到\ ``extern "C" {}``\ 中。


小结\ ``extern "C"``
~~~~~~~~~~~~~~~~~~~~

通过上面的分析，我们知道\ ``extern "C"``\ 的作用是实现\ **C和C++的混合编程**\ 。

在C++源文件中的语句前面加上\ ``extern "C"``\ ，表明它按照C的编译和链接规约来编译和链接，而不是C++的编译和链接规约。
这样，在类C的代码中就可以调用C++的函数or变量等。


C和C++相互调用
--------------

我们既然知道\ ``extern "C"``\ 是实现C和C++的混合编程，下面就分别介绍如何在C++中调用C的代码，C中调用C++的代码。

首先要明白C和C++相互调用，得知道它们之间的编译和链接差异，及如何使用\ ``extern "C"``\ 来实现相互调用。

<跳转到函数名修饰规则>


C++中调用C的代码
~~~~~~~~~~~~~~~~

假设一个C的头文件\ *cHeader.h*\ 中包含一个函数\ ``print(int i)``\ ，为了在C++中能够调用它，必须加上\ ``extern``\ 关键字。它的代码如下:

.. code-block:: c

    #ifndef C_HEADER
    #define C_HEADER

    extern void print(int i);

    #endif  // C_HEADER

相对应的实现文件\ *cHeader.c*\ 的代码为:

.. code-block:: c

    #include <stdio.h>

    void print(int i)
    {
        printf("cHeader %d\n", i);
    }

在C++代码中引用C中的\ ``print(int i)``\ 函数:

.. code-block:: c++

    extern "C"{
    #include "cHeader.h"
    }

    int main(int argc, char** argv)
    {
        print(3);
       
        return 0;
    }


C中调用C++的代码
~~~~~~~~~~~~~~~~

例如，在\ *cppHeader.hpp*\ 头文件中定义了下面的代码:

.. code-block:: c++

    #ifndef CPP_HEADER
    #define CPP_HEADER

    extern "C" void print(int i);

    #endif  // CPP_HEADER

相对应的实现文件\ *cppHeader.cpp*\ 文件中代码如下:

.. code-block:: c++

    #include "cppHeader.hpp"

    #include <iostream>

    using namespace std;

    void print(int i)
    {
        cout << "cppHeader: " << i << endl;
    }

在C的代码\ *c.c*\ 中调用\ ``print``\ 函数:

.. code-block:: c

    extern void print(int i);

    int main(int argc, char** argv)
    {
        print(3);
       
        return 0;
    }


C和C++混合调用特别之处 -- 函数指针
----------------------------------

当C和C++混合编程时，有时候会用一种语言定义函数指针，而在应用中将函数指针指向另一种语言定义的函数。
如果C和C++共享同一种编译和链接，函数调用机制，这样做是可以的。
然而，这样的通用机制，通常不能假定它存在，因此我们必须小心地确保函数以期望的方式调用。
当指定一个函数指针的编译和链接方式时，函数的所有类型，包括函数名，函数引入的变量也按照指定的方式编译和链接。

例如:

.. code-block:: c++

    typedef int (*FT) (const void*, const void*);  // style of C++

    extern "C"{
        typedef int (*CFT) (const void*, const void*);  // style of C
        void qsort(void* p, size_t n, size_t sz, CFT cmp);  // style of C
    }

    void isort(void* p, size_t n, size_t sz, FT cmp);  // style of C++
    void xsort(void* p, size_t n, size_t sz, CFT cmp);  // style of C

    // style of C
    extern "C" void ysort(void* p, size_t n, size_t sz, FT cmp);

    int compare(const void*, const void*);  // style of C++
    extern "C" ccomp(const void*, const void*);  // style of C

    void f(char* v, int sz)
    {
        // Error, as qsort is style of C
        // but compare is style of C++
        qsort(v, sz, 1, &compare);
        qsort9(v, sz, 1, &ccomp);

        isort(v, sz, 1, &compare);  // ok

        // Error, as isort is style of C++
        // but ccomp is style of C
        isort(v, sz, 1, &ccomp);
    }

注意，\ ``typedef int (*FT)(const void*, const void*)``\ ，表示定义了一个函数指针的别名\ *FT*\ ，这种函数指针指向的函数有这样的特征:
返回值为int型，有两个参数，参数类型可以为任意类型的指针(因为是\ ``void *``)。

最典型的函数指针的别名的例子是，信号处理函数\ ``signal``\ ，它的定义如下:

.. code-block:: c

    typedef void(*HANDLER)(int);
    HANDLER signal(int, HANDLER);

上面的代码定义了信号处理函数\ ``signal``\ ，它的返回值类型为\ ``HANDLER``\ ，有两个参数分别为int，HANDLER。这样就避免了要这样定义signal函数:

.. code-block:: c

    void (*signal (int, void(*)(int)))(int)

比较之后就可以明显的体会到\ ``typedef``\ 的好处。

--------------

原文地址: `C++项目中的extern "C" {} <https://www.cnblogs.com/skynet/archive/2010/07/10/1774964.html>`_

