__cplusplus
===========

``__cplusplus``\ 是C++编译器中定义的一个宏，可以通过判断是否定义了这个宏，来判断当前的编译器是是C编译器还是C++编译器。

这个宏主要用来解决C/C++混合编程中的问题，一般用法如下:

.. code-block:: c++

    #ifdef __cplusplus
    extern "C" {
    #endif
    ....
    #ifdef __cplusplus
    }
    #endif

``__cplusplus``\ 宏是某一个被定义的值，在不同版本的C++编译器中，其值不同:

.. code-block:: c++

    C++03: __cplusplus = 199711L
    C++11: __cplusplus = 201103L
    C++14: __cplusplus = 201402L

可以根据\ ``__cplusplus``\ 宏的值，来判断当前使用的C++版本:

.. code-block:: c++

    #if __cplusplus < 201103L
    #error "Shoud use -std=c++11 option for compile"
    #endif

