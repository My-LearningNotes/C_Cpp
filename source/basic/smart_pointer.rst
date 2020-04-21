C++智能指针
===========


概述
----

有的编程语言, 如Java和Golang有自动垃圾回收机制, 可以管理分配的堆内存, 在对象失去引用时自动回收.

C++中没有自动垃圾回收机制, 必须自己去释放分配的堆内存, 否则就会造成内存泄漏.

通常, 对于动态内存管理, 用\ ``new``\ 从堆上申请内存, 使用完后, 需要使用\ ``delete``\ 释放申请的内存.
如果因为某种原因(比如程序员忘记了, 或者不小心删除了delete语句)没有释放, 则会造成\ **内存泄漏(memory leak)**\ .
还有一些情况, 即使没有忘记delete语句, 也可能会有内存泄漏.

比如, 当异常和动态内存管理一起使用时:

.. code-block:: cpp

    void remodel(std::string &str)
    {
        std::string &ps = new std::string(str);
        ...
        if (weird_thing())
            throw exception();
        str = *ps;
        delete ps;
        return;
    }

在上面的代码中, 如果发生了异常, \ ``delete``\ 将不会执行, 因此导致内存泄漏.

一种解决方法是:\ **在引发异常的函数中捕获该异常, 在catch块中执行一些清理工作, 之后再抛出该异常**\ , 这种方法是可行的, 但不是最好的方法.

.. code-block:: cpp

    void remodel(std::string &str)
    {
        std::string *ps = new std::string(str);
        ...
           
        try {
            if (weird_thing())
           	    throw exception();
        }
        catch (exception &ex) {
            delete ps;
            throw;
        }
       
        str = *ps;
        delete ps;
        return;
    }

在C++中, 对于动态内存管理, 有更灵巧的解决方法: \ **智能指针**\ .

.. note::

    **智能指针是在C++11中引入的, 智能指针是行为类似指针的模板类, 用智能指针管理动态分配的内存, 在智能指针的生命周期结束时, 在析构时会自动释放其管理的堆内存.**


-   使用智能指针, 需要包含相应的头文件\ ``memory``\ , 智能指针位于命名空间\ ``std``\ 中;

-   共有四个智能指针模板类: ``auto_ptr``\ , \ ``unique_ptr``\ , \ ``shared_ptr``\ 和\ ``weak_ptr``\ ;

    .. note::

        ``auto_ptr``\ 是在C++98中引入的, 已逐步放弃, 所以尽量不要再使用.

-   智能指针是模板类, 因此, 在实例化时, 需要先具体化, 即用模板的语法指定具体的类型;

    智能指针模板类的构造函数是用\ ``explicit``\ 声明的, 所以, 不允许隐式的转换.

    Example:

    .. code-block:: cpp

        shared_ptr<double> pd;
        double *p_reg = new double;
        pd = p_reg;  // not allowed(implicit conversion)
        pd = shared_ptr<double> (p_reg);  // allowed(explicit conversion)
        shared_ptr<double> pshared = p_reg;  // not allowed(explicit conversion)
        shared_ptr<double> pshared(p_reg);  // allowed(explicit conversion)

        shared_ptr<int> p1 = make_shared<int>(int(10));  // make_shared函数
        unique_ptr<int> p2 = make_shared<int>(int(20));  // make_unique函数

-   智能指针对象的使用, 在很多方面都类似于常规指针;

    -   可以使用解除引用运算符(\ ``*p``);

    -   访问结构体成员(\ ``p->buffIndex``);

    -   赋给同类型的常规指针;

    -   赋给同类型的另一个智能指针对象.

-   智能指针智能用来管理堆内存, 即用\ ``new``\ 动态分配的内存, 应该避免用于非动态管理的内存, 否则会造成错误;

    Example:

    .. code-block:: cpp

        string vacation("I wandered lonely as a cloud.");
        shared_ptr<string> pvan(&vacation);  // No! 将智能指针用于非堆内存

-   如果有两个指针指向同一个对象, 则可能会产生问题.

    比如, 程序可能会试图释放一个对象两次.
    对于智能指针也是一样, 智能指针会在其生命周期结束时, 在其析构函数中释放其管理的对象.
    如果有两个智能指针指向同一个对象, 则该对象可能会被释放两次.

    要避免这种问题, 有多种方法:

    -   对于智能指针的赋值, 定义赋值运算符, 使其执行\ **深赋值**\ ;

        这样, 两个指针将指向不同的对象, 其中的对象是另一个对象的副本.

    -   ``auto_ptr``\ 和\ ``unique_ptr``\ 使用的是独占所有权策略.

        对于一个特定的对象, 只能有一个智能指针可拥有它, 只有拥有该对象的智能指针的析构函数会释放对象.

        赋值操作会转让所有权.

        ``unique_ptr``\ 的策略比\ ``auto_ptr``\ 更加严格.

        对于\ ``auto_ptr``\ 类型的智能指针, 可以将一个指针赋值给另一个指针, 同时转交所有权, 但这会产生\ **悬空的指针**\ , 可能会产生运行时的错误.

        Example:

        .. code-block:: cpp

            auto_ptr<string> p1(new string("hello, world."));
            auto_ptr<string> p2;
            p2 = p1;  // 将p1赋值给p2，同时转交所有权，之后p1不再指向原来的对象，再使用p1来访问对象时会产生错误

        也就是说, 对于\ ``auto_ptr``\ 类型的智能指针, 赋值操作转交所有权后, 会产生悬空的指针, 可能会造成运行时的错误.

        ``unique_ptr``\ 和\ ``auto_ptr``\ 一样采用所有权策略, 但\ ``unique_ptr``\ 比\ ``auto_ptr``\ 更加安全.
        ``unique_ptr``\ 不允许产生悬空的指针, 否则会在编译阶段报错.

        Example:

        .. code-block:: cpp

            unique_ptr<string> p3(new string("hello, world"));
            unique_ptr<string> p4;
            p4 = p3;  // 编译时报错，赋值后，p3悬空，这是unique_ptr不允许的

-   ``shared_ptr``\ 是更加智能的指针, 它会跟踪引用特定对象的智能指针数, 这称为\ **引用计数(reference counting)**\ .

    赋值时, 计数加1; 而指针过期时, 计数减1.
    仅当最后一个指针过期时, 才释放其管理的动态内存.


详细介绍
--------


``auto_ptr``
~~~~~~~~~~~~~

``auto_ptr``\ 在C++98引入, 定义在头文件\ ``memory``\ , 其功能和用法类似于\ ``unique_ptr``\ .
为何从C++11开始, 引入\ ``unique_ptr``\ 来替代\ ``auto_ptr``\ 呢?

原因主要有以下几点: 

-   基于安全的考虑

    先来看下面的赋值语句：

    .. code-block:: cpp

        auto_ptr<string> p(new string("I reigned lonely as a clound"));
        auto_ptr<string> vocation;
        vocation = p;

    上述赋值语句将完成什么工作呢?
    如果\ ``p``\ 和\ ``vocation``\ 是常规指针, 则两个指针将指向同一个\ ``string``\ 对象. 
    这是不能接受的, 因为程序将试图删除同一个对象两次, 一个是\ ``p``\ 过期时, 一次是\ ``vocation``\ 过期时. 
    
    要避免这种问题, 方法有多种: 

    -   定义赋值运算符/复制构造函数, 使之执行深赋值.

        这样, 两个指针将指向不同的对象, 其中一个对象是另一个对象的副本, 缺点是浪费空间, 所以智能指针都未采用此方案.

   -    建立所有权(ownership)概念.

        对于特定的对象, 只能有一个智能指针可拥有, 这样只有拥有对象的智能指针的析构函数会删除该对象. 
        然后让赋值操作转让所有权, 这就是\ ``auto_ptr``\ 和\ ``unique_ptr``\ 的策略, 但\ ``unique_ptr``\ 的策略更严格.

   -    创建智能更高的指针, 跟踪引用特定对象的智能指针数, 这称为引用计数.

       例如, 赋值时, 计数加1, 而指针过期时, 计数减1. 
       当引用计数为0时才调用delete, 这就是\ ``shared_ptr``\ 采用的策略.

    
        下面举个例子来说明:

        .. code-block:: cpp

            #include <iostream>
            #include <string>
            #include <memory>

            using namespace std;

            int main()
            {
                auto_ptr<string> films[5] = {
                    auto_ptr<string> (new string("A")),
                    auto_ptr<string> (new string("B")),
                    auto_ptr<string> (new string("C")),
                    auto_ptr<string> (new string("D")),
                    auto_ptr<string> (new string("E"))
                };

                auto_ptr<string> p;
                p = films[2];  // films[2] loses owership

                for (int i = 0; i < 5; ++i)
                    cout << *films[i] << endl;

                cout << *p << endl;

                return 0;
            }

        运行发现程序崩溃, 因为\ ``films[2]``\ 已经是空指针了.

        但如果这里把\ ``auto_ptr``\ 换成\ ``shared_ptr``\ 或\ ``unique_ptr``\ 后, 程序就不会崩溃了, 原因如下:

        * 使用\ ``shared_ptr``\ 时运行正常, 因为\ ``shared_ptr``\ 采用引用计数, \ ``p``\ 和\ ``films[2]``\ 都指向同一块内存, 在释放空间时因为要先判断引用计数值的大小, 因此不会出现多次删除同一个对象的错误;

        * 使用\ ``unique_ptr``\ 时编译出错, 与\ ``auto_ptr``\ 一样, \ ``unique_ptr``\ 也采用所有权模型, 但在使用\ ``unique_ptr``\ 时, 程序不会等到运行阶段崩溃, 而是在编译期就报错.

        .. note::

            这就是为何要摒弃\ ``auto_ptr``\ 的原因, 一句话总结就是: \ **避免因潜在的内存问题导致程序崩溃**\ .

从上面可见, \ ``unique_ptr``\ 比\ ``auto_ptr``\ 更加安全, 因为\ ``auto_ptr``\ 有拷贝语义, 拷贝后原对象变得无效了, 再次访问原对象时会导致程序崩溃; 
而\ ``unique_ptr``\ 则禁止了拷贝语义, 但提供了移动语义, 即可以使用\ ``std::move()``\ 进行控制权的转移.

Example:

.. code-block:: cpp

    unique_ptr<string> p1(new string("A"));
    unique_ptr<string> p2(p1);  // 编译时报错, 已禁止拷贝
    unique_ptr<string> p3 = p1;  // 编译时报错, 已禁止赋值
    unique_ptr<string> p4 = std::move(p1);  // 控制权转移

    auto_ptr<string> p5(new string("B"));
    auto_ptr<string> p6(p5);  // 编译通过
    auto_ptr<string> p7 = p5;  // 编译通过

这里要注意, 在使用\ ``std::move``\ 将\ ``unique_ptr``\ 的控制权转移后, 不能够再通过\ ``unique_ptr``\ 来访问和控制资源了, 否则同样会出现程序崩溃. 
我们可以在使用\ ``unique_ptr``\ 访问资源之前, 使用成员函数\ ``get()``\ 进行判空操作.

.. code-block:: cpp

    unique_ptr<string> p2 = std::move(p1);  // 控制权转移
    if (p1.get() != nullptr)  // 判空操作更安全
    {
        // do something
    }

-   ``unique_ptr``\ 不仅安全, 而且灵活

    如果\ ``unique_ptr``\ 是个临时右值, 编译器允许拷贝语义.

    .. code:: cpp

        unique_ptr<string> demo(string s)
        {
            unique_ptr<string> temp(new string(s));
            return tmp;
        }

        unuque_ptr<string> p;
        p = demo("Hello, world");

    ``demo``\ 返回一个临时的\ ``unique_ptr``\ , 然后\ ``p``\ 接管了临时对象\ ``unique_ptr``\ 所管理的资源, 
    而返回时临时的\ ``unique_ptr``\ 被销毁, 也就是说没有机会使用临时的\ ``unique_ptr``\ 来访问无效的数据, 
    换句话来说, 这种赋值时不会出现任何问题的, 即没有理由禁止这种赋值. 实际上, 编译器确实允许这种赋值. 
    相对于\ ``auto_ptr``\ 任何情况下都允许拷贝语义, 这正是\ ``unique_ptr``\ 更加灵活的地方.

-   扩展\ ``auto_ptr``\ 不能完成的功能

    -   ``unique_ptr``\ 可放在容器中, 弥补了\ ``auto_ptr``\ 不能作为容器元素的缺点;

    -   管理动态数组, 因为\ ``unique_ptr``\ 有\ ``unique_ptr<X[]>``\ 重载版本, 销毁动态对象时调用\ ``delete[]``\ ;

    -   自定义资源删除操作(Deleter).

        ``unique_ptr``\ 默认的资源删除操作时\ ``delete/delete []``\ , 若需要, 可以进行自定义.

    综上所述, 基于\ ``unique_ptr``\ 的安全性和扩充的功能, 我们应该使用\ ``unique_ptr``\ 取代\ ``auto_ptr``\ .


``unique_ptr``
~~~~~~~~~~~~~~

<<<<<<< HEAD
``unique_ptr``\ 在C++11引入, 旨在替代不安全的\ ``auto_ptr``\ .
=======
>>>>>>> 35a565f... 增加关于C++强制类型转换的笔记

``unique_ptr``\ 持有对对象的独有权, 两个\ ``unique_ptr``\ 不能指向同一个对象，即\ ``unique_ptr``\ 不能共享它所管理的对象. 
它无法复制到其它的\ ``unique_ptr``\ , 无法通过值传递到函数, 也无法用于需要副本的任何标准模板库(STL)算法. 
智能移动\ ``unique_ptr``\ , 即对资源管理权限可以实现转移. 
这意味着, 内存资源所有权可以转移到另一个\ ``unique_ptr``\ , 并且原始\ ``unique_ptr``\ 不再拥有此资源. 
实际使用中, 建议将对象限制为由一个所有者所有, 因为多个所有权会使程序逻辑变得复杂. 
因此, 当需要智能指针用于存C++对象时, 可使用\ ``unique_ptr``\ .

``unique_ptr``\ 与原始指针一样有效, 并可用于STL容器. 
将\ ``unique_ptr``\ 实例添加到STL容器运行效率很高, 因为通过\ ``unique_ptr``\ 的移动构造函数, 不再需要进行复制操作. 
``unique_ptr``\ 指针与其所指对象的关系: 在智能指针生命周期内, 可以改变智能指针所指对象, 
如创建智能指针时通过构造函数指定, 通过\ ``reset``\ 方法重新指定, 通过\ ``release``\ 方法释放所有权, 通过移动语义转移所有权, \ ``unique_ptr``\ 还可能没有对象, 这种情况被称为empty.

``unique_ptr``\ 的基本操作有:

.. code-block:: cpp

    /* unique_ptr的基本操作 */

    // 创建
    unique_ptr<int> p1;  // 创建空的智能指针
    p1.reset(new int(3));  // 绑定动态对象
    uique_ptr<int> p2(new int(4));  // 创建时指定动态对象


    // 所有权的变化
    int *p = p2.release();  // 释放所有权
    unique_ptr<string> p3(new string("abc"));
    unique_ptr<string> p4 = std::move(p3);  // 所有权转移(通过移动语义), p3所有权转移后, 变成"空指针"
    p4.reset(p3.release());  // 所有权转移
    p4 = nullptr;  // 显式销毁所指对象, 同时智能指针变为空指针, 与p4.reset()等价


``shared_ptr``
~~~~~~~~~~~~~~


``shared_ptr``\ 简介
^^^^^^^^^^^^^^^^^^^^

``shared_ptr``\ 是一个标准的共享所有权的智能指针, 允许多个指针指向同一个对象.

``shared_ptr``\ 最初实现于Boost库中, 后在C++11引入到C++ STL.
``shared_ptr``\ 利用引用计数的方式实现了对所管理的对象的所有权的分享, 即允许多个\ ``shared_ptr``\ 共同管理一个对象.

``shared_ptr``\ 是为了解决\ ``auto_ptr``\ 在对象所有权上的局限性(\ ``auto_ptr``\ 是独占的), 在使用引用计数的机制上提供了可以共享所有权的智能指针, 当然这需要额外的开销:

-   ``shared_ptr``\ 对象除了包括一个所有用对象的指针外, 还必须包括一个引用计数代理对象的指针;

-   时间上的开销主要在初始化和拷贝操作上, \ ``*``\ 和\ ``->``\ 操作符重载的开销跟`auto_ptr是一样的;

<<<<<<< HEAD
-  开销并不是我们不使用\ ``shared_ptr``\ 的理由, 永远不要进行不成熟的优化, 直到性能分析器告诉你这一点.


``weak_ptr``
~~~~~~~~~~~~


``weak_ptr``\ 简介
^^^^^^^^^^^^^^^^^^^

``weak_ptr``\ 被设计为与\ ``shared_ptr``\ 共同工作, 可以从一个\ ``shared_ptr``\ 或者另一个\ ``weak_ptr``\ 对象构造而来. 
``weak_ptr``\ 是为了配合\ ``shared_ptr``\ 而引入的一种智能指针, 它更像是\ ``shared_ptr``\ 的一个助手而不是智能指针, 
因为它不具有普通指针的行为, 没有重载\ ``operator*``\ 和\ ``operator->``\ , 因此取名为\ *weak*\ , 表明其是功能较弱的智能指针. 
它的最大作用在于协助\ ``shared_ptr``\ 工作, 可获得资源的观测权, 像旁观者那样观测资源的使用情况. 
观察者意味着\ ``weak_ptr``\ 只对\ ``shared_ptr``\ 进行引用, 而不改变其引用计数, 当被观察的\ ``shared_ptr``\ 失效后, 相应的\ ``weak_ptr``\ 也相应失效.


``weak_ptr``\ 用法
^^^^^^^^^^^^^^^^^^^

使用\ ``weak_ptr``\ 的成员函数\ ``use_count()``\ 可以观测资源的引用计数, 
另一个成员函数\ ``expired()``\ 的功能等价于\ ``use_count == 0``\ , 但更快, 表示被观测的资源(也就是\ ``shared_ptr``\ 管理的资源)已经不复存在. 
\ ``weak_ptr``\ 可以使用一个非常重要的成员函数\ ``lock()``\ 从被观测的\ ``shared_ptr``\ 获得一个可用的\ ``shared_ptr``\ 管理的对象, 从而操作资源. 
但当\ ``expired() == true``\ 时, \ ``lock()``\ 函数将返回一个存储空指针的\ ``shared_ptr``\ .
=======

weak_ptr
~~~~~~~~

.. _header-n224:

weak_ptr简介
^^^^^^^^^^^^

``weak_ptr``\ 被设计为与\ ``shared_ptr``\ 共同工作，可以从一个\ ``shared_ptr``\ 或者另一个\ ``weak_ptr``\ 对象构造而来。\ ``weak_ptr``\ 是为了配合\ ``shared_ptr``\ 而引入的一种智能指针，它更像是\ ``shared_ptr``\ 的一个助手而不是智能指针，因为它不具有普通指针的行为，没有重载\ ``operator*``\ 和\ ``operator->``\ ，因此取名为\ *weak*\ ，表明其是功能较弱的智能指针。它的最大作用在于协助\ ``shared_ptr``\ 工作，可获得资源的观测权，像旁观者那样观测资源的使用情况。观察者意味着\ ``weak_ptr``\ 只对\ ``shared_ptr``\ 进行引用，而不改变其引用计数，当被观察的\ ``shared_ptr``\ 失效后，相应的\ ``weak_ptr``\ 也相应失效。

.. _header-n228:

weak_ptr用法
^^^^^^^^^^^^

使用\ ``weak_ptr``\ 的成员函数\ ``use_count()``\ 可以观测资源的引用计数，另一个成员函数\ ``expired()``\ 的功能等价于\ ``use_count == 0``\ ，但更快，表示被观测的资源(也就是\ ``shared_ptr``\ 管理的资源)已经不复存在。\ ``weak_ptr``\ 可以使用一个非常重要的成员函数\ ``lock()``\ 从被观测的\ ``shared_ptr``\ 获得一个可用的\ ``shared_ptr``\ 管理的对象，从而操作资源。但当\ ``expired() == true``\ 时，\ ``lock()``\ 函数将返回一个存储空指针的\ ``shared_ptr``\ 。
>>>>>>> 35a565f... 增加关于C++强制类型转换的笔记

总的来说，\ ``weak_ptr``\ 的基本用法总结如下：

.. code-block:: cpp

    // weak_ptr基本用法总结

    weak_ptr<T> w;  // 创建空weak_ptr, 可以指向类型为T的对象
    weak_ptr<T> w(sp);  // 与shared_ptr指向相同的对象, shared_ptr引用计数不变, T必须能转换为sp指向的类型
    w = p;  // p可以是shared_ptr或weak_ptr, 赋值后w与p共享独享
    w.reset();  // 将w置空
    w.use_count();  // 返回与w共享对象的shared_ptr的数量
    w.expired();  // 若w.use_count 为0, 返回true, 否则返回false
    w.lock();  // 如果expired()为true, 返回一个空的shared_ptr, 否则返回非空shared_ptr

下面是一个简单的示例：

.. code-block:: cpp

    #include <assert.h>
    #include <iostream>
    #include <memory>
    #include <string>

    using namespace std;

    int main()
    {
        shared_ptr<int> sp(new int(10));
        assert(sp.use_count() == 1);
        weak_ptr<int> wp(sp);  // 从shared_ptr创建weak_ptr
        assert(wp.use_count() == 1);
        if (!wp.expired())  // 判断weak_ptr观察的对象是否失效
        {
            shared_ptr<int> sp2 = wp.lock();  // 获得一个shared_ptr
            *sp2 = 100;
            assert(wp.use_count() == 2);
        }
        assert(sp.use_count() == 1);
        cout << "int: " << *sp << endl;

        return 0;
    }

程序输出为：100

从上面可以看到, 尽管以\ ``shared_ptr``\ 来构造\ ``weak_ptr``\ , 但是\ ``weak_ptr``\ 并没有改变\ ``shared_ptr``\ 的引用计数.


``weak_ptr``\ 的作用
^^^^^^^^^^^^^^^^^^^^

``weak_ptr``\ 到底有什么用呢?

从上面的那个例子来看, 似乎没有任何作用. 
其实\ ``weak_ptr``\ 可用于打破循环引用. 
引用计数是一种便利的内存管理机制, 但它有一个很大的缺点, 那就是不能管理循环引用的对象.


如何选择智能指针
----------------

在了解STL的四种智能指针之后，我们可能会想到另一个问题: 在实际应用中，应使用哪种智能指针呢?

下面给出几个使用指南:

-   如果程序中要使用多个指向同一个对象的指针, 应选择\ ``shared_ptr``\ , 这样的情况包括:

    -   将指针作为参数或者函数的返回值进行传递, 应该使用\ ``shared_ptr``;

    -   两个对象都包含指向第三个对象的指针, 此时应该使用\ ``shared_ptr``\ 来管理第三个对象;

    -   STL容器包含指针.

        很多STL算法都支持复制和赋值操作, 这些操作可用于\ ``shared_ptr``\ , 但不能用于\ ``unique_ptr``\ (编译时错误)和\ ``auto_ptr``\ (运行时错误).

-   如果程序不需要多个指向同一个对象的指针, 则可以使用\ ``unique_ptr``\ . 

    如果函数使用\ ``new``\ 分配内存, 并返回指向该内存的指针看, 将其返回类型声明为\ ``unique_ptr``\ 是不错的选择. 
    这样, 所有权转让给接收返回值的\ ``unique_ptr``\ , 而该智能指针将负责调用\ ``delete``\ . 
    可将\ ``unique_ptr``\ 存储到STL容器中, 只要不调用将\ ``unique_ptr``\ 复制或赋值给另一个的算法(如\ ``sort()``).

Example:

.. code-block:: cpp

    #include <algorithm>
    #include <iostream>
    #include <memory>
    #include <vector>

    using namespace std;

    unique_ptr<int> make_int(int n)
    {
        return unique_ptr<int>(new int(n));
    }

    void show(unique_ptr<int> &p)
    {
        cout << *p << " ";
    }

    int main()
    {
        vector<unique_ptr<int>> vp(10);
        for (int i = 0; i < vp.size(); ++i)
            vp[i] = make_int(rand() % 1000);  // copy temporary unique_ptr
        vp.push_back(make_int(rand() % 1000));  // ok, because arg is temporary
        for_each(vp.begin(), vp.end(), show);  // use for_each()

        return 0;
    }

其中\ ``push_back``\ 调用没有问题, 因为它的参数是一个临时\ ``unique_ptr``\ , 该\ ``unique_ptr``\ 被赋给vp中的一个\ ``unique_ptr``\ . 
另外, 如果按值而不是按引用给\ ``show()``\ 传递对象, \ ``for_each()``\ 将非法, 因为这将导致使用一个来自vp的非临时的\ ``unique_ptr``\ 初始化\ ``show()``\ 的参数, 而这是不允许的.

在\ ``unique_ptr``\ 为右值时, 可将其赋值给\ ``shared_ptr``\ , 这与将一个\ ``unique_ptr``\ 赋给另一个\ ``unique_ptr``\ 需要满足的条件相同, 即\ ``unique_ptr``\ 必须是一个临时对象. 
模板\ ``shared_ptr``\ 包含一个显式构造函数, 可用于将右值\ ``unique_ptr``\ 转换为\ ``shared_ptr``\ .
``shared_ptr``\ 将接管原来归\ ``uniqur_ptr``\ 所有的对象.

-   虽然说在满足\ ``unique_ptr``\ 要求的条件时, 使用\ ``auto_ptr``\ 也可以完成对内存资源的管理, 但是因为\ ``auto_ptr``\ 不够安全, 不提倡使用, 即任何情况下都不应该使用\ ``auto_ptr``\ .

-   为了解决\ ``shared_ptr``\ 的循环引用问题，我们可以使用\ ``weak_ptr``\ .

--------------

参考：

`C++ STL四种智能指针 <https://blog.csdn.net/k346k346/article/details/81478223>`__
