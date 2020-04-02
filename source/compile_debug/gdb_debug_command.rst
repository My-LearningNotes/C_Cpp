GDB常用调试命令
===============


简介
----

GDB是Linux 上最常见的调试工具。
为了使我们的可执行程序能够被GDB调试，需要在编译时加上调试信息，也就是加上\ ``-g``\ 选项:

.. code-block:: shell
    :emphasize-lines: 1

    gcc -g example.c -o example

``-gn``\ 选项表示在编译程序时加入调试信息，n(= 1, 2, 3)表示调试等级，其中\ ``-g``\ 表示\ ``-g2``\ 。

调试信息会增加程序的大小，有时候大小差异会高达10倍之多，所以一般在发布应用程序时是不会以\ ``-g``\ 参数编译的。
我们可以在事后使用\ ``strip``\ 命令清除掉应用程序中的调试信息:

.. code-block:: shell
    :emphasize-lines: 1

    strip example

.. note::
    
    如果是交叉编译的程序，需要使用交叉编译工具链中的strip命令。


frame和stack的概念
------------------

函数调用由连续栈帧组成。每个栈帧记录一个函数调用的信息，这些信息包括函数参数，函数变量，函数运行地址。

当程序启动后，栈中只有一个帧，这个帧就是main函数的帧。我们把这个帧叫做初始化帧或者叫做最外层帧。

每当一个函数被调用，一个新帧将被建立，每当一个函数返回，函数帧将被移除。
如果函数是个递归函数，栈中将有很多帧是记录同一个函数的。

当前执行的函数的帧被称作最深帧，这个帧是现存栈中最近被创建的帧。

gdb为所有存活的栈帧分配一个数字编号，最深帧的编号为0，调用它的那个帧的编号是1，以此类推。


常用调试命令
------------

-   ``help(h)``

    显示指令的简短说明。

    例如，\ ``help breakpoint``\ 。

-   ``file [program]``

    加载调试程序，等同于\ ``gdb [program]``\ 。

-   ``run(r)``

    执行程序，或是从头再执行程序，程序会在第一个断点停下来。

-   ``start``

    执行程序，程序会在main函数的第一条语句处停下来。

-   ``kill``

    中止程序的执行。

-   ``backtrace(bt)``

    堆栈追踪。

    会显示出上层所有的frame的简略信息。

-   ``print(p)``

    打印变量的值。

    例如，\ ``print i``\ ，打印变量i的值。

-   ``list(l)``

    打印源代码。

    若在编译时没有加上\ ``-g``\ 参数，\ ``list``\ 指令将没有作用。

-   ``whatis``

    打印变量的类型。

    例如，\ ``whatis i``\ ，打印变量i的类型。

-   ``breakpoint(b, bre, break)``

    设置断点。

    -  使用\ ``info breakpoint(info b)``\ 来查看已设定了哪些断点。

    -  在程序被中断之后，使用\ ``info line``\ 来查看正停在哪一行

-   ``continue(c, cont)``

    继续执行，和\ ``breakpoint``\ 搭配使用。

-   ``frame``

    显示正在执行的行数，函数名称，以及所传递的参数等frame信息。

-   ``next(n)``

    单步执行，遇到函数时不会进入函数单步执行，而是会跳过函数的执行过程。

-   ``step(n)``

    单步执行，遇到函数时会进入函数中单步执行。

-   ``until``

    直接跑完一个while循环。

-   ``return``

    中止当前函数的执行过程，并立即返回。类似于C里的return。

-   ``finish``

    跳过当前函数的执行过程，立即执行完整个函数。

-   ``up``

    直接跳到上一层的frame，并显示栈信息，如进入点及传入的参数等。

-   ``down``

    直接跳到下一层的frame；必须使用up回到上层的frame后，才能使用down跳回来。

-   ``display``

    在每一步执行后，自动显示某个变量的值。

-   ``undisplay``

    取消display某个变量。

-   ``commands``

    在遇到断点时要自动执行的指令。

-   ``info``

    显示一些特定的信息。

    如：\ ``info break``\ ，显示断点；\ ``info share``\ ，显示共享函数库信息。

-   ``disable``

    关闭某个\ ``breakpoint``\ 或\ ``watchpoint``\ 的功能。

-   ``enable``

    将被\ ``disable``\ 暂时关闭的功能再启用。

-   ``clear/delete``

    删除某个breakpoint/watchpoint。

-   ``set``

    设定特定参数。

    如：\ ``set env``\ ，设定环境变量。也可以用来修改变量的值。

-   ``unset``

    取消特定参数。

    如：\ ``unset env``\ ，删除环境变量。

-   ``show``

    显示特定参数。

    如：\ ``show environment``\ ，显示环境变量。

-   ``attach`` PID

    载入已执行中的程序以进行调试。其中的PID可由\ ``ps``\ 指令取得。

-   ``detach`` PID

    释放已attach的程序。

-   ``shell``

    执行shell指令。

    例如：\ ``shell ls``\ ，呼叫shell以执行\ ``ls``\ 指令。

-   ``quit``

    退出gdb。

-   ``<Enter>``

    重复执行上个指令。


示例
----

我们以下面的程序为例，说明常用的GDB调试命令。

.. code-block:: c

    // example.c

    #include <stdio.h>

    long func(int a)
    {
        long sum = 0;
        for (int j = 1; j <= a; j++)
        {
     	    sum += j;
        }

        return sum;
    }

    int main()
    {
        int a = 100;
        long sum = func(a);
        printf("%ld\n", sum);

        return 0;
    }


-   **编译程序**

    ``gcc -g example.c -o example``

-   **启动gdb**

    可以通过\ ``gdb program``\ 来启动gdb，其中\ ``program``\ 是要调试的程序；
    也可以直接执行\ ``gdb``\ ，之后执行\ ``file program``\ 来加载要调试的程序。

    启动之后，就可以看到命令提示符\ ``(gdb)``\ 了：

    .. code-block:: shell
        :emphasize-lines: 16

        GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
        Copyright (C) 2018 Free Software Foundation, Inc.
        License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
        This is free software: you are free to change and redistribute it.
        There is NO WARRANTY, to the extent permitted by law. Type "show copying"
        and "show warranty" for details.
        This GDB was configured as "x86_64-linux-gnu".
        Type "show configuration" for configuration details.
        For bug reporting instructions, please see:
        <http://www.gnu.org/software/gdb/bugs/>.
        Find the GDB manual and other documentation resources online at:
        <http://www.gnu.org/software/gdb/documentation/>.
        For help, type "help".
        Type "apropos word" to search for commands related to "word"...
        Reading symbols from example...done.
        (gdb) 

-   **查看源码**

    载入要调试的程序之后，在gdb的命令行中输入\ ``list``\ 或其简写\ ``l``\ ，可以查看到程序的源码以及行号，默认会显示10行源码，按回车之后会显示接下来的10行，直到文件的末尾。

    .. code-block:: shell
        :emphasize-lines: 1, 12

        (gdb) list
        2
        3   long func(a)
        4	{
        5	long sum = 0;
        6	for (int j = 1; j <= a; j++)
        7	{
        8	sum += j;
        9	}
        10
        11	return sum;
        (gdb)
        12	}
        13
        14	int main()
        15	{
        16	int a = 100;
        17	long sum = func(a);
        18	printf("%ld\n", sum);
        19
        20	return 0;
        21	}
        (gdb)

-   **添加断点**

    在gdb下添加断点使用命令\ ``break``\ 或简写\ ``b``\ ，有下面几个常见用法，具体可以执行\ ``help break``\ 查看详细用法。

    -  ``break 函数名``

    -  ``break 行号``

    -  ``break 文件名:函数名``

    -  ``break 文件名:行号``

    -  ``break 函数名/行号 if条件``  // 程序运行时，只有满足给定的条件，才设置该断点

    比如我们在main函数和func函数上各添加一个断点:

    .. code-block:: shell
        :emphasize-lines: 1, 3

        (gdb) break main
        Breakpoint 1 at 0x685: file example.c, line 16.
        (gdb) break func
        Breakpoint 2 at 0x651: file example.c, line 5.
        (gdb) 

    如上，我们成功加上了两个断点，在正确加上断点之后，会对应有一行输出，告诉我们断点的内存地址，断点对应的源文件名和行号。

-   **查看断点**

    在加上断点之后，我们可以通过\ ``info break``\ 命令断点的信息：

    .. code-block:: shell
        :emphasize-lines: 1

        (gdb) info break
        Num Type       Disp Enb Address            What
        1   breakpoint keep y   0x0000000000000685 in main at example.c:16
        2   breakpoint keep y   0x0000000000000651 in func at example.c:5
        (gdb)

*   **禁用和解禁断点**

    通过\ ``disable <break number>``\ 来禁用指定Num的断点，如下我们禁用1号断点:

    .. code-block:: shell
        :emphasize-lines: 1, 2

        (gdb) disable 1
        (gdb) info break
        Num Type       Disp Enb Address            What
        1   breakpoint keep n   0x0000000000000685 in main at example.c:16
        2   breakpoint keep y   0x0000000000000651 in func at example.c:5
        (gdb)

    如上，\ ``disable 1``\ 之后，断点1的\ ``Enb``\ 列由之前的\ ``y``\ 变成了\ ``n``\ ，说明断点1已被禁用。

    通过\ ``enable <break number>``\ 可以来解禁断点，如下我们对刚才禁用的断点1解禁:

    .. code-block:: shell
        :emphasize-lines: 1, 2

        (gdb) enable 1
        (gdb) info break
        Num Type       Disp Enb Address            What
        1   breakpoint keep y   0x0000000000000685 in main at example.c:16
        2   breakpoint keep y   0x0000000000000651 in func at example.c:5
        (gdb)

    如上，断点1的Enb列又变成y了，它被成功解禁。

-   **删除断点**

    可以用\ ``delete <break number>``\ 命令来删除掉一个断点，如下我们删除断点1:

    .. code-block:: shell
        :emphasize-lines: 1, 2

        (gdb) delete 1
        (gdb) info break
        Num Type       Disp Enb Address            What
        2   breakpoint keep y   0x0000000000000651 in func at example.c:5
        (gdb)

    如上，断点1被成功删除。

-   **启动程序**

    可以使用\ ``run``\ 命令或者简写\ ``r``\ 来启动程序的执行，程序启动后，遇到断点就会停下来:

    .. code-block:: shell
        :emphasize-lines: 1

        (gdb) run
        Starting program: /home/sylar/example
        Breakpoint 2, func (a=100) at example.c:5
        5	long sum = 0;
        (gdb)

    如上，程序执行到断点2的时候就停止执行了。

    还可以使用\ ``start``\ 命令来启动程序，不同的是，用\ ``start``\ 启动程序后，程序会停在main函数的第一条语句处：

    .. code-block:: shell
        :emphasize-lines: 1

        (gdb) start
        Temporary breakpoint 1 at 0x685: file example.c, line 18.
        Starting program: /tmp/example
        Temporary breakpoint 1, main () at example.c:18
        18	int a = 100;

-   **查看变量的值**

    ``print <variable name>``/``p <variable name>``\ 可以查看某一个变量的当前值：

    .. code-block:: shell
        :emphasize-lines: 1

        (gdb) print sum
        $1 = 0
        (gdb) 

    如上，当前sum的值为0。

-   **单步执行**

    ``next``\ 命令或者\ ``n``\ 可以单步执行，如果遇到函数，跳过函数的执行过程。

    .. code-block:: shell

        Starting program: /tmp/example
        Breakpoint 1, main () at example.c:18
        18	int a = 100;
        (gdb) next
        19	long sum = func(a);
        (gdb) next
        20	printf("%ld\n", sum);
        (gdb)

-   **跳入跳出函数**

    ``next``\ 单步执行，如果遇到函数，会跳过函数的执行过程；

    如果想跳入到函数的内部，可以使用\ ``step``\ 命令获取简写\ ``s``\ ，如果想跳出函数的执行过程可以使用\ ``finish``\ 指令，
    这时会导致函数执行完毕，并且打印出一些函数的返回信息，并且程序停在函数后的第一条语句处。

    .. code-block:: shell
        :emphasize-lines: 1, 3, 7, 10

        (gdb) break 19
        Breakpoint 1 at 0x68c: file example.c, line 19.
        (gdb) run
        Starting program: /tmp/example
        Breakpoint 1, main () at example.c:19
        19	long sum = func(a);
        (gdb) step
        func (a=100) at example.c:7
        7	long sum = 0;
        (gdb) finish
        Run till exit from #0 func (a=100) at example.c:7
        0x0000555555554696 in main () at example.c:19
        19	long sum = func(a);
        Value returned is $1 = 5050
        (gdb)

-   **追踪堆栈**

    使用\ ``backtrace(bt)``\ 指令可以追踪堆栈。

    .. code-block:: shell
        :emphasize-lines: 1

        (gdb) bt
        \#0 func (a=100) at example.c:8
        \#1 0x0000555555554696 in main () at example.c:19
        (gdb)

    ``#0``\ (最上面的栈帧)表示当前的函数调用，以此类推。

-   **查看传递给函数的参数**

    使用\ ``info args``\ ，可以查看调用当前函数时，传递的参数。

-   **查看当前栈帧中的所有局部变量的值**

    使用\ ``info locals``\ 可以查看当前函数中所有局部变量的值。

-   **监控变量**

    使用\ ``watch <variable name>``\ 命令可以实现监控变量，使用\ ``info watch``\ 命令可以查看监控的有哪些变量。
    同时，\ ``break``\ 所拥有的\ ``enable``, ``disable``, ``delete``\ 等动词对于\ ``watch``\ 依然适用，且用法大同小异。

    ``watch``\ 和\ ``break``\ 的不同:

    * ``break``\ 是在某个位置设置断点，当程序运行到这个位置时就会停下来；

    * ``watch``\ 是监控某个变量的值，当这个变量的值发生变化时，程序就会停下来，并且打印监控的变量变化前后的值。

    .. code-block:: shell
        :emphasize-lines: 1, 6, 8, 11

        (gdb) start
        Temporary breakpoint 1 at 0x685: file example.c, line 18.
        Starting program: /tmp/example 
        Temporary breakpoint 1, main () at example.c:18
        18	int a = 100;
        (gdb) watch sum
        Hardware watchpoint 2: sum
        (gdb) info watch
        Num Type Disp Enb Address What
        2 hw watchpoint keep y sum
        (gdb) c
        Continuing.
        Hardware watchpoint 2: sum
        Old value = 0
        New value = 5050
        main () at example.c:20
        20	printf("%ld\n", sum);
        (gdb)

-   **显示变量的值**

    使用\ ``display <variable name>``\ 命令可以在每一步执行之后，打印变量的当前值。

    .. code-block:: shell
        :emphasize-lines: 1, 6, 8, 11, 14, 18

        (gdb) start
        Temporary breakpoint 1 at 0x685: file example.c, line 18.
        Starting program: /tmp/example
        Temporary breakpoint 1, main () at example.c:18
        18	int a = 100;
        (gdb) display sum
        1: sum = 0
        (gdb) n
        19	long sum = func(a);
        1: sum = 0
        (gdb) n
        20	printf("%ld\n", sum);
        1: sum = 5050
        (gdb) n
        5050
        22	return 0;
        1: sum = 5050
        (gdb) n
        23	}
        1: sum = 5050
        (gdb)

    如果不想显示某个变量的值，可以使用\ ``undisplay <variable name>``\ 。

-   **进入shell/执行shell指令**

    -  ``shell``\ 命令可以让我们从gdb命令行环境进入到shell的命令行环境，当我们在shell命令行环境中输入\ ``exit``\ 退出后，我们就又回到了之前的gdb命令行环境了；

    -  也可以以\ ``shell cmd``\ 的形式直接执行shell命令，其中\ ``cmd``\ 是shell中的指令，例如：\ ``shell ls``\ ，呼叫shell以执行\ ``ls``\ 指令。

-   **可视化调试**

    在gdb命令行环境中输入\ ``wi``\ 命令，可以让我们进入可视化调试环境，这个环境可以看到源码，所使用的调试命令与上面讲到的一致。

--------------

参考:

`GDB介绍 <[https://b8807053.pixnet.net/blog/post/336154079-%5B%E8%BD%89%E8%B2%BC%5Dgdb-%E4%BB%8B%E7%B4%B9](https://b8807053.pixnet.net/blog/post/336154079-[轉貼]gdb-介紹)>`__

`gcc/g++常用编译选项和gdb常用调试命令 <https://andrewpqc.github.io/2018/11/25/gcc-and-gdb/>`__

