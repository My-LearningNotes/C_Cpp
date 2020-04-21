C++四种强制类型转换
===================


C风格的强制类型转换
-------------------

C风格的强制类型转换(Type Cast)使用下面的形式:

.. code-block:: c
    :emphasize-lines: 1

    Type b = (Type)a;

C++也是支持C风格的强制类型转换的.
此外, 为了形式上类似于函数调用, C++中的强制类型转换通常使用下面的形式:

.. code-block:: cpp
    :emphasize-lines: 1

    Type b = Type(a);

但是, C风格的强制类型转换可能带来一些隐患, 让一些问题难以察觉, 所以C++提供了一组可以用在不同场合的\ **强制类型转换函数**\ :

* ``const_cast`` 

* ``static_cast``

* ``dynamic_cast``

* ``reinterpret_cast``

这四个强制类型转换都是\ **模板函数**\ , 所以需要:

-  在\ ``<>``\ 中指定转换的目标类型

-  在()中指定被转换的原始对象


C++四种强制类型转换函数
-----------------------

``const_cast``
~~~~~~~~~~~~~~

-  ``const_cast``\ 用于修改常量指针/常引用, 将其转换为\ **非const**\ 的.

Example:  

    .. code-block:: cpp
        :emphasize-lines: 19

        #include <iostream>

        using namespace std;

        int main()
        {
            // 原始数组
            int array[4] = {1, 2, 3, 4};
            // 打印原始数据
            for (int i = 0; i < 4; i++)
                cout << array[i] << " ";
            cout << endl;

            // 常量指针
            const int *p1 = array;
            // p1[0]1 = 100;  // Error

            // 通过const_cast将常量指针转换为非常量指针
            int *p2 = const_cast<int *>(p1);
            // 修改数据
            for (int i = 0; i < 4; i++)
                p2[i] += 1;

            // 打印修改后的数据
            for (int i = 0; i < 4; i++)
                cout << array[i] << " ";
            cout << endl;

            return 0;
        }

需要注意, 在C++中, const类型的变量被视为常量, 编译器在编译期就会将所有对const变量的引用用值替换. 
所以, 使用\ ``const_cast``\ 对const类型的变量的指针或引用转换, 并不能达到预期的结果.

Example:

    .. code-block:: cpp

        #include <iostream>

        using namespace std;

        int main()
        {
            const int a = 100;

            const int *p1 = &a;
            int *p2 = const_cast<int *>(p1);
            *p2 = 200;

            cout << a << endl;

            return 0;
        }

运行上面的代码, 我们可能会期望输出结果为200, 但实际的输出结果是100.

原因就是: a被声明为const类型的变量, 在编译期, 编译器就会将所有引用a的地方用其值100替换.


``static_cast``
~~~~~~~~~~~~~~~

-  ``static_cast``\ 作用和C语言风格的强制类型转换的效果基本一样, 由于没有运行时类型检查来保证转换的安全性, 所以这类型的强制转换和C语言风格的强制类型转换都有安全隐患;

-  用于类层次结构中基类和派生类之间指针或引用的转换;

    .. note::

        注意，进行向上转换(把派生类的指针或引用转换成基类表示)是安全的；
        进行向下转换(把基类的指针或引用转换为派生类表示)时，由于没有动态类型检查，所以是不安全的。

-  用于基本数据类型之间的转换;

   例如, 把int转换为char, 把int转换为enum, 这种转换的安全性需要开发者来维护.

-  在C++ Primer中说到: C++的任何的隐式类型转换都是使用\ ``static_cast``\ 来实现的.


Example:

    .. code-block:: cpp

        // 基本数据类型之间的转化
        float f_pi = 3.1415927f;
        int i_pi = static_cast<int>(f_pi);  // i_pi的值为3

        // 类的层次结构中，基类和派生类之间指针或引用的转换
        class Base
        {
            ...
        };

        class Derived : public Base
        {
            ...
        };

        // 派生类向基类的转换
        // 编译通过，是安全的
        Derived derived;
        Base *base = static_cast<Base *>(&derived);

        // 基类向派生类的转换，不安全的
        // 编译通过，但不是安全的
        Base base;
        Derived *derived = static_cast<Derived *>(&base);	


``dynamic_cast``
~~~~~~~~~~~~~~~~

``dynamic_cast``\ 应该是这个四种中最特殊的一个, 因为它涉及到面向对象的多态性和程序运行时的状态, 也与编译器的属性设置有关, 所以不能完全使用C语言的强制类型转换替换, 它也是最常用的, 最不可缺少的一种强制类型转换.


``reinterpret_cast``
~~~~~~~~~~~~~~~~~~~~

``reinterpret_cast``\ 用来处理无关类型的转换.

它是用在任意的指针之间的转换, 引用之间的转换, 指针和足够大的int型之间的转换, 整数到指针的转换.

IBM C++对\ ``reinterpret_cast``\ 推荐使用的地方:

-  A pointer to any integral type large enough to hold it.

-  A value of integral or enumeration type to pointer.

-  A pointer to a function to a pointer to a function of a different
   type.

-  A pointer to an object to a pointer to an object of a different type.

-  A pointer to a member to a pointer to a member of a different class
   or type, if the types of the members are both function types or
   object types.


总结
----

在使用强制类型转换的时候, 要先考虑清楚是否真的需要强制类型转换和应该使用哪种强制类型转换.

