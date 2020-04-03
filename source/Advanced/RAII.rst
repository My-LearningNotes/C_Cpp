RAII原理
========

什么是RAII
----------

RAII(Resource Acquisition Is Initialization)是由C++之父Bjarne Stroustrup提出的设计理念，中文翻译为资源获取即初始化，
他说: **使用局部对象来管理资源的技术称为资源获取即初始化**\ ，其核心思想是\ **把资源和局部对象的生命周期绑定，对象创建时获取资源，对象销毁时释放资源**\ ，
从而避免依靠程序员手动的去释放资源(因为程序员可能会无意间忘记，从而导致资源泄漏)。

这里的资源主要是指操作系统中有限的东西，如内存，网络套接字等等，局部对象是指存储在栈的对象，它的生命周期是由操作系统来管理的，无需人工介入。


RAII的原理
----------

资源的使用一般经历三个步骤：

- 获取资源

- 使用资源

- 销毁资源
  
资源的销毁往往是程序员经常忘记的一个环节，所以最好能够在程序中让资源自动销毁。

C++之父给出了解决问题的方案：RAII，它充分利用了C++语言局部对象自动销毁的特性来控制资源的生命周期。

以下面的代码示例说明:

.. code-block:: cpp
    :emphasize-lines: 13, 18, 38

    // example.cpp

    #include <iostream>

    using namespace std;

    class Person
    {
    public:
        Person(const string name = "", int age = 0) :
            name_(name), age_(age)
        {
            cout << "Init a person!" << endl;
        }

        ~Person()
        {
            cout << "Destory a person!" << endl;
        }

        const string &getName() const
        {
            return this->name_;
        }

        int getAge() const
        {
            return this->age_;
        }

    private:
        const string name_;
        int age_;
    };

    int main()
    {
        Person person;

        return 0;
    }

编译程序并运行:

.. code-block:: shell

   g++ example.cpp -o example
   ./example

运行结果:

.. code-block:: shell

   Init a person!
   Destory a person!

可以看到，当我们在main函数中声明一个局部对象的时候，会自动调用构造函数对对象进行初始化，当整个main函数执行完成后，自动调用析构函数来销毁对象，整个过程无需人工介入，由操作系统自动完成。
于是，很自然联想到，当我们在使用资源的时候，在构造函数中进行初始化，在析构函数中进行销毁。

整个RAII过程可以总结为四个步骤：

-  **设计一个类封装资源**

-  **在构造函数中初始化**

-  **在析构函数中执行销毁操作**

-  **使用时声明一个该对象的类**


RAII的应用
----------

这里通过一个简单的例子来说明如何将RAII应用到我们的代码中。

Linux下经常会使用多线程技术，说到多线程，就得提到互斥锁，
互斥锁主要用于互斥，互斥是一种\ **竞争关系**\ ，用来保护临界资源只被一个线程访问，
按照我们前面的分析，我们封装一下POSIX标准的互斥锁。

.. code-block:: cpp
    :emphasize-lines: 10, 11, 13, 14, 38, 39, 40, 41, 43, 44, 45, 46

    // mutex.h

    #include <pthread.h>
    #include <cstdlib>
    #include <stdio.h>

    class Mutex
    {
    public:
        Mutex();
        ~Mutex();

        void Lock();
        void Unlock();

    private:
        pthread_mutex_t mu_;

        // No copying
        Mutex(const Mutex&);
        void operator=(const Mutex&);
    };

    // mutex.cpp

    #include "mutex.h"

    #include <string.h>

    static void PthreadCall(const char *label, int result)
    {
        if (result != 0)
        {
            fprintf(stderr, "pthread %s: %s\n", label, strerror(result));
        }
    }

    Mutex::Mutex()
    {
        PthreadCall("init mutex", pthread_mutex_init(&mu_, NULL));
    }

    Mutex::~Mutex()
    {
        PthreadCall("destroy mutex", pthread_mutex_destroy(&mu_));
    }

    void Mutex::Lock()
    {
        PthreadCall("lock", pthread_mutex_lock(&mu_));
    }

    void Mutex::Unlock()
    {
        PthreadCall("unlock", pthread_mutex_unlock(&mu_));
    }



写到这里，其实就可以使用Mutex来锁定临界区，
但我们发现Mutex只是用来对锁的初始化和销毁，我们还得在代码中调用\ ``Lock``\ 和\ ``Unlock``\ 函数，这又是一个对立操作，
所以我们可以继续使用RAII进行封装，代码如下：

.. code-block:: cpp
    :emphasize-lines: 8, 9, 10, 11, 12, 14, 15, 16, 17

    // mutexlock.h

    #include "mutex.h"

    class MutexLock
    {
    public: 
        explicit MutexLock(Mutex *mu)
            : mu_(mu)
        {
            this->mu_->Lock();
        }
       
        ~MutexLock()
        {
            this->mu_->Unlock();
        }
       
    private:
        Mutex *const mu_;
        // No copying allowed
        MutexLock(const MutexLock&);
        void operator=(const MutexLock&);
    }
    
到这里我们就真正封装了互斥锁，下面我们通过一个简单的例子来使用它，代码如下：

.. code-block:: cpp
    :emphasize-lines: 15

    // main.cpp

    #include <unistd.h>
    #include <iostream>

    #include "mutexlock.h"

    #define NUM_THREADS 10000

    int num = 0;
    Mutex mutex;

    void *count(void *args)
    {
        MutexLock lock(&mutex);
        num++;
    }

    int main()
    {
        int t;
        pthread_t thread[NUM_THREADS];

        for (t = 0; t < NUM_THREADS; t++)
        {
            int ret = pthread_create(&thread[t], NULL, count, NULL);
            if (ret)
            {
                return -1;
            }
        }

        for (t = 0; t < NUM_THREADS; t++)
        {
            pthread_join(thread[t], NULL);
        }
        std::cout << num << std::endl;

        return 0;
    }

编译程序并运行：

.. code-block:: shell

    g++ mutex.cpp main.cpp -lpthread -o main
    ./main

运行结果: 1000，符合预期。

--------------

参考：

`c++经验之谈一：RAII原理介绍 <https://zhuanlan.zhihu.com/p/34660259>`__
