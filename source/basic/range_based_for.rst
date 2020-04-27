C++11新特性 --- 基于范围的for循环
=================================


简介
----

如果要遍历一个数组, 传统的实现方式是使用\ ``for``\ 循环和一个整型变量.

Example:

    .. code-block:: cpp

        int array[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        for (int i = 0; i < 10; ++i)
        {
   	        std::cout << array[i] << std::endl;
        }

遍历容器类, 可以使用\ ``for``\ 循环和迭代器.

Example:

    .. code-block:: cpp

        std::vector<int> vec{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        for (auto it = vec.begin(); it != vec.end(); ++it)
        {
   	        std::cout << *it << std::endl;
        }


上面的两种方法, 都必须明确的指定for循环的开头以及结尾条件.

而在Python中存在一种for的使用方法, 不需要明确给出循环的开头和结尾条件, 就可以遍历整个容器.
C++11中引入了这种方法, 就是\ **基于范围的for(Range-Based_for)**\ .

基于范围的for循环的一般格式:

.. code-block:: cpp

    // 依次将array/container中的各个值传递给rangeVariable
    for (dataType rangeVariable : array/container)
    {
        statement;
    }

.. tip::

    遍历变量的类型可以声明为\ ``auto``\ , 让编译器自动推断类型.


用基于范围的for循环改写上面两个例子:

.. code-block:: cpp

    int array[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    for (auto i : array)
    {
   	    std::cout << i << std::endl;
    }

    std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    for (auto i: vec)
    {
   	    std::cout << i << std::endl;
    }

可以看到, 改写后的使用方法简单了很多, 代码的可读性也提升了一个档次. 
但是需要注意的是, 上述对容器的遍历是以\ **值传递**\ 的方式进行的, 所以不会修改容器的元素.

如果在遍历时需要修改元素的值, 方法就是将遍历的变量声明为引用类型.

Example:

    .. code-block:: cpp

        std::vector<int> vec {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

        std::cout << "修改前:" << std::endl;
        for (auto &i : vec)
        {
   	        std::cout << i++ << std::endl;
        }
        std::cout << "修改后:" << std::endl;
        for (auto i : vec)
        {
   	        std::cout << i << std::endl;
        }

 
使用基于范围的for循环一定要注意这一点: **遍历变量默认是值传递的, 声明为引用类型时是引用传递.**


基于范围的for循环使用时需要注意的细节
-------------------------------------

虽然基于范围的for循环使用起来非常的方便, 不用再关注for的开始条件和结束条件了, 但是还是有一些细节问题在使用的时候需要注意.

-   来看看基于范围的for对于容器map的遍历

Example:

    .. code-block:: cpp

        std::map<string, int> map = {{"a", 1}, {"b", 2}, {"c", 3}};
        for (auto &val : map)
        {
   	        std::cout << val.first << "->" << val.second << std::endl;
        }

为什么是使用\ ``val.first``\ 和\ ``val.second``\ 而不是直接输出value呢?

在遍历容器的时候, auto自动推导的类型是容器的value_type类型, 而不是迭代器, 
而map中的value_type是\ ``std::pair``, 也就是说val的类型是\ ``std::pair``\ , 因此需要使用\ ``val.first``, ``val.second``\ 来访问数据.

-   使用基于范围的for循环还要注意一些容器类本身的约束
  
比如set容器内的元素本身由容器特性就决定了其元素是只读的, 哪怕使用了引用类型来遍历set元素, 也是不能修改容器元素的.

Example:

    .. code-block:: cpp

        std::set<int> ss = {1, 2, 3, 4, 5};
        for (auto &n: ss)
        {
            std::cout << n++ << std::endl;
        }

上述代码定义了一个set, 使用引用类型遍历set元素, 然后对元素的值进行修改.

这段代码在编译时就会报错, 因为set容器中的元素是只读的.

同样, map中的first元素也是不能进行修改的.

-   再来看看如果冒号后面的表达式不是一个容器而是一个函数, 看看函数会被调用多少次?

Example:

    .. code-block:: cpp

        std::set<int> ss = {1, 2, 3, 4, 5};
        const std::set<int> &getSet()
        {
   	        return ss;
        }

        int main()
        {
   	        for (auto &n: getSet())
            {
   		        std::cout << n << std::endl;
   		    }

   	        return 0;
        }

虽然冒号后面的是一个函数调用, 但是该函数返回的是一个容器.
程序执行时, 先执行该函数, 返回一个容器, 然后和在冒号后直接写一个容器是一样的.

-   基于范围的for循环和迭代器一样, 在迭代时不能对容器进行修改, 否则会产生错误

Example:

    .. code-block:: cpp

        std::vector<int> vec = {1, 2, 3, 4, 5};

        int main()
        {
   	        for (auto &n : vec)
   	        {
   		        std::cout << n << std::endl;
   		        vec.push_back(6);
   	        }
        }

上述代码在遍历vector时, 在容器中插入一个元素6, 运行上述代码时会程序会崩溃.
究其原因, 是由于在遍历容器时, 在容器中插入一个元素导致迭代器失效.

因此, **基于范围的for循环和普通的for循环一样, 在遍历的过程中如果修改容器会造成迭代器失效, 从而引发错误.**

.. note::

    注意\ **修改容器**\ 和\ **修改容器中的元素**\ 的区别: 

    *   修改容器, 是指调用了\ ``push_back``\ , \ ``pop_back``\ 等方法, 对容器的大小(容器中元素的个数)进行了修改;

    *   修改容器中的元素, 是指没有对容器大小进行修改, 只是修改了容器中元素的值.


自定义的类实现基于范围的for循环
-------------------------------

Todo

--------------

参考:

https://blog.csdn.net/hailong0715/article/details/54172848
