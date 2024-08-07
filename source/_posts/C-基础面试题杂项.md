---
title: C++基础面试题杂项
date: 2024-06-21 16:27:12
tags: C++
categories:
    C++基础
password: mzy666
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# C/C++

## 常见问题：智能指针、多态、虚函数、STL 底层原理。

1. 什么是C++多态

> C++的多态是面向对象的重要特性之一，具体就是**同一个接口，使用不同的实例而执行不同操作**，包含静态多态和动态多态
>
> 1. 静态多态是通过函数重载和模板实现的，指在编译期就决定了调用哪个函数，根据参数列表决定
> 2. 动态多态是通过虚函数实现的，在运行时根据对象确定具体调用哪一个类的虚函数

2. 智能指针

> 智能指针是通过RAII技术对普通指针进行封装，更好的管理指针，防止内存泄漏、指针悬空等问题
>
> 在最开始没有智能指针的时候，使用普通指针经常会出现二次释放，内存泄漏等问题，而智能指针的引入就是为了解决这些问题，通过RAII技术对普通指针进行封装，当智能指针所指对象的生存周期结束后，会在析构函数中释放掉申请的内存，从而防止内存泄漏。（栈对象）

3. **你在定义函数接口时，如何设计参数类型以提高传递效率？**

> 可以使用引用来传递参数，这样不会拷贝对象，节约内存和事件。如果传递的是临时对象，可以使用右值引用`&&`和`std::move`来避免拷贝

4. 右值引用了解吗

> 右值引用就是对右值的引用，给右值取别名，主要作用是延长对象的生命周期，可以减少拷贝，节省内存，从而提高效率

5. 程序执行的四个阶段？模板用过吗，是在哪个阶段

> 静态多态是通过函数重载和模板实现的，指在编译期就决定了调用哪个函数，根据参数列表决定

6. 动态链接和静态链接了解吗？

> 静态链接：由链接器再链接时将库的内存加入到可执行程序中
>
> - 独立性强：生成的可执行文件包含所有依赖，不需要在运行时额外的库文件。
> - 兼容性好：运行时不依赖于系统中安装的库版本，不会遇到“库版本冲突”问题。
> - 文件体积大：每个可执行文件都包含完整的库代码，导致文件体积增大。
> - 更新麻烦：如果库有更新，需要重新编译所有使用该库的程序。
>
> 动态链接：把链接的过程延迟到运行时再进行，不嵌入到可执行文件中，把库加载进内存，就一份
>
> - 文件体积小：可执行文件不包含库代码，只包含对库的引用。
> - 易于更新：更新库不需要重新编译程序，只需替换库文件。
> - 内存效率高：多个程序可以共享同一个库文件的内存实例，减少内存使用。
> - 依赖性强：可执行文件在运行时需要能够找到并加载正确版本的库文件。
> - 兼容性问题：库文件版本不匹配可能导致程序运行失败。

7. 虚拟内存是怎么转化到物理内存的

> 页表：想要把虚拟内存地址，映射到物理内存地址，最直观的办法，就是来建一张映射表。这个映射表，能够实现虚拟内存里面的页，到物理内存里面的页的映射。这个映射表，在计算机里面，就叫作页表。
>
> 1. 把虚拟内存地址，切分成页号和偏移量的组合；
> 2. 从页表里面，查询出虚拟页号，对应的物理页号；
> 3. 直接拿物理页号，加上前面的偏移量，就得到了物理内存地址。

8. 为什么静态函数不能是虚函数

9. **没有`this`指针**：静态函数没有`this`指针，而虚函数依赖于`this`指针来实现动态绑定。没有`this`指针，就无法访问vtable，也就无法实现虚函数的多态性。
10. **与对象无关**：静态函数与具体对象无关，而虚函数的目的是针对对象实现多态性。这两者的设计目的和使用场景不同，无法兼容。
11. **类和实例的区别**：静态函数是类级别的，而虚函数是实例级别的。虚函数需要对象的实例来决定调用哪个具体的函数实现，而静态函数不依赖对象实例。



> 自我介绍及项目经验简要介绍自身背景、技术技能及项目经验。
>
> C++多态编译时多态：函数重载、模板。运行时多态：通过虚函数和虚表（vtable）实现。
>
> 虚函数重写使用override关键字重写虚函数。纯虚函数必须重写，非纯虚函数可以不加override。
>
> 运行时多态实现原理通过虚表指针找到相应函数地址进行调用。
>
> 高性能数据库项目描述项目功能、用途和主要挑战（如LSM树数据结构的实现）。
>
> 多线程操作单线程访问，使用线程锁保护数据一致性。
>
> 智能指针unique_ptr：单一所有者。shared_ptr：多个所有者，引用计数。weak_ptr：防止shared_ptr循环引用。
>
> 内存管理智能指针不能完全保证内存不会泄漏（循环引用问题）。
>
> 函数参数设计使用引用或右值引用提高传递效率，减少不必要的拷贝。

1. 除了多态和继承还有什么面向对象方法

   1. **封装**是将对象的状态（属性）和行为（方法）隐藏在对象内部，并通过公共接口（公共方法）与外部进行交互。这样可以保护对象的内部状态不被外部直接访问和修改，从而提高代码的安全性和维护性。
   2. **抽象**是从具体的实例中提取出共有的特征，将不必要的细节隐藏起来。通过抽象，可以创建更加通用和简洁的模型。
   3. **组合**是通过将对象包含在其他对象内部来构建复杂对象的过程。聚合是较松散的一种对象组合关系，表示“整体-部分”的关系。

2. C++内存分布。什么样的数据在栈区，什么样的在堆区

   1. **栈区**用于存储局部变量、函数参数和返回地址。栈区的内存分配由编译器自动管理，具有自动回收的特性。当函数调用时，局部变量和函数参数会被压入栈中；当函数返回时，这些数据会被弹出栈。
      - **局部变量**：在函数内部声明的变量。

      - **函数参数**：传递给函数的参数。

      - **返回地址**：用于保存函数调用的返回地址。

   2. **堆区**用于动态分配内存，生命周期由程序员控制。堆区的内存分配和释放需要使用 `new` 和 `delete` 运算符。堆区适合用于需要在运行时决定大小并且生命周期不局限于单个函数调用的内存。

      - **动态分配的对象和数组**：使用 `new` 和 `delete` 运算符分配和释放的内存。

   3. **全局区**用于存储全局变量和静态变量。全局变量在整个程序生命周期内有效。静态变量在它们所属的作用域内有效，但它们的生命周期是整个程序运行期间。

      - **全局变量**：在所有函数之外声明的变量。
      - **静态变量**：使用 `static` 关键字声明的变量，无论是在函数内部还是类内部。
      - **静态成员变量**：在类中使用 `static` 关键字声明的静态成员变量。

   4. **常量区**用于存储常量数据和未初始化的全局/静态变量。常量区的数据在程序运行期间是只读的。

      - **字符串字面量**：使用双引号括起来的字符串。
      - **const 变量**：用 `const` 关键字声明的变量。

   5. **代码区**用于存储程序的可执行代码，包括函数和方法的机器指令。这些指令在程序加载时被操作系统加载到内存中，并且通常是只读的。

   ![image-20240612122221519](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240612122221519.png)

3. C++内存管理（RAII啥的）

4. C++从源程序到可执行程序的过程

   1. 预处理：	将宏定义展开（**预处理过程主要处理那些源文件中的以“#”开始的预编译指令**）
   2. 编译：   将源文件编译为汇编语言（编译过程就是把预处理的文件进行一系列的**词法分析**，**语法分析**，**语义分析**以及**优化后产生相应的汇编代码文件**。）
   3. 汇编：   将汇报文件转换为机器码，生成可重定位目标文件
   4. 链接：将多个目标文件及所需要的库连接成最终的可执行目标文件。

5. 一个对象=另一个对象会发生什么（赋值构造函数）

6. 如果new了之后出了问题直接return。会导致内存泄漏。怎么办
   （智能指针，raii，stl容器）

7. 多进程fork后不同进程会共享哪些资源

   **文件描述符**：父子进程共享文件描述符所指向的文件表项。这意味着对文件描述符的操作（如读写文件指针的移动）会相互影响。

   **文件锁**：父子进程之间共享文件锁。

   **信号处理程序**：父子进程共享信号处理程序的设置。

   **消息队列、信号量和共享内存**：如果父进程使用了这些进程间通信机制，子进程将共享这些资源。

8. 多线程里线程的同步方式有哪些

   1. 互斥锁是一种用于保护临界区的同步机制，它确保同一时刻只有一个线程能够进入临界区执行代码，从而避免多个线程同时访问共享资源而导致的数据竞争和错误。互斥锁通过 `pthread_mutex_init()、pthread_mutex_lock() 和 pthread_mutex_unlock() `等函数来进行操作。
   2. 读写锁允许多个线程同时读取共享资源，但只允许一个线程写入共享资源。这种机制适用于读操作频繁而写操作较少的场景，可以提高程序的并发性能。读写锁通过 `pthread_rwlock_init()、pthread_rwlock_rdlock() 和 pthread_rwlock_wrlock() `等函数来进行操作。
   3. 条件变量是一种用于线程间等待和通知的机制，允许线程等待某个条件达成后再继续执行。条件变量通常与互斥锁配合使用，以确保线程在等待和通知的过程中能够安全地访问共享资源。条件变量通过 `pthread_cond_init()、pthread_cond_wait() 和 pthread_cond_signal() `等函数来进行操作。
   4. 在线程间通信中，信号量通常用于控制对临界资源的访问，类似于进程间通信中的信号量。线程间信号量可以用于实现互斥和同步，确保多个线程之间的协同工作。信号量通过 s`em_init()、sem_wait() 和 sem_post() `等函数来进行操作。
   5. 原子操作

9. size_of是在编译期还是在运行期确定

   在C++中，`sizeof`运算符的计算是在编译期确定的，而不是在运行期确定的。`sizeof`用于获取类型或对象的大小（以字节为单位），并且其结果在编译时就已经固定，不会在运行时再进行计算。

10. 函数重载的机制。重载是在编译期还是在运行期确定

    函数重载是C++中的一种静态多态性，通过允许在同一作用域内定义多个同名但参数列表不同的函数来实现。重载机制在编译期由编译器确定，而不是在运行期确定。

11. 指针常量和常量指针

    > 常量指针:常量指针指向一个常量值，这意味着通过这个指针不能修改所指向的对象。指针本身可以改变，指向不同的对象。
    >
    > ```c++
    > int x = 10;
    > int y = 20;
    > const int *ptr = &x;  // ptr 是一个常量指针，可以改变指向对象，但不能通过 ptr 修改 x 的值
    > 
    > ptr = &y;  // 合法，ptr 可以指向不同的对象
    > *ptr = 30; // 非法，不能通过 ptr 修改 y 的值
    > ```
    >
    > 指针常量：指针常量是一个指针变量，它被初始化后就不能改变其指向的对象，但通过这个指针可以修改对象的值。
    >
    > ```c++
    > int x = 10;
    > int y = 20;
    > int *const ptr = &x;  // ptr 是一个指针常量，不能改变指向，但可以通过 ptr 修改 x 的值
    > 
    > ptr = &y;   // 非法，ptr 不能指向不同的对象
    > *ptr = 30;  // 合法，可以通过 ptr 修改 x 的值
    > ```

12. vector的原理，怎么扩容

    `std::vector` 是 C++ 标准库中的一个动态数组容器，它可以自动调整大小以适应元素的增加或减少。其底层实现依赖于动态分配内存，并在需要时进行扩容。

    >**`std::vector` 的扩容机制**
    >
    >当 `std::vector` 需要增加元素且当前容量不足时，它会执行以下步骤进行扩容：
    >
    >1. **分配新的内存块**：`std::vector` 会分配一个更大的内存块。新容量一般是旧容量的两倍，以平衡扩容次数和内存使用效率。
    >2. **复制元素**：将旧内存块中的所有元素复制到新的内存块中。这涉及到调用元素的复制构造函数。
    >3. **释放旧内存块**：释放旧的内存块，避免内存泄漏。
    >4. **更新内部指针**：更新内部指针以指向新的内存块，并调整容量信息。

13. 介绍一下const

    在 C++ 中，`const` 关键字用于定义常量或者防止变量被修改。它可以应用于变量、指针、成员函数等，以增加代码的安全性和可读性。

14. 引用和指针的区别

    1. 指针可以为空，如指向`nullptr`；引用不能为空
    2. 指针可以改变所指的对象，引用初始化后不能被改变
    3. 指针是一个实体，由内存的；引用仅是个别名

15. RAII基于什么实现的（生命周期、作用域、构造析构）

    RAII 基于 C++ 对象的生命周期、作用域以及构造函数和析构函数来实现。

16. 手撕：Unique_ptr，控制权转移(移动语义）手撕：类继承，堆栈上分别代码实现多态

17. 右值引用

    **左值（Lvalue）**：表示一个有名字的变量或内存中的一个位置，可以取其地址。例如，变量 `x`、数组元素 `arr[i]`、解引用指针 `*ptr` 等。

    **右值（Rvalue）**：表示一个临时值或不具名的对象，通常是表达式的结果，不能取其地址。例如，字面值 `42`、表达式 `x + y` 的结果等。

    **右值引用（rvalue reference）**：使用 `&&` 表示，允许你绑定到右值。右值引用主要用于实现**移动语义**和**完美转发**。

18. 函数参数可不可以传右值

    **右值引用（rvalue reference）**：使用 `&&` 表示，允许你绑定到右值。右值引用的主要目的是支持移动语义和避免拷贝开销。通过右值引用，函数参数可以直接接收右值，从而实现高效的资源管理。

19. 参考c/c++堆栈实现自己的堆栈。要求：不能用stl容器。

20. stl容器了解吗？底层如何实现：vector数组，map红黑树，红黑树的实现

21. 完美转发介绍一下 去掉std::forward会怎样？

    完美转发的目标是保证函数模板在传递参数时，不改变参数的类型特性。例如，如果传递的是左值，则保持左值；如果传递的是右值，则保持右值。

    1. **万能引用（Universal Reference）**：函数模板参数类型 `T&&` 可以同时匹配左值和右值。

    2. **`std::forward`**：用于保持参数的类型特性进行转发。

22. 介绍一下unique_lock和lock_guard区别？

    1. `std::lock_guard`：

       `std::lock_guard` 是一个简单的、轻量级的 RAII（Resource Acquisition Is Initialization）锁管理类。它在构造时锁定互斥锁，在析构时解锁。`std::lock_guard` 不能显式地锁定或解锁互斥锁，也不能在作用域内转移锁的所有权。

    2. `std::unique_lock`

       `std::unique_lock` 是一个更灵活的锁管理类，提供了比 `std::lock_guard` 更多的功能。它允许显式地锁定和解锁，延迟锁定，尝试锁定，甚至在作用域内转移锁的所有权。

23. C代码中引用C++代码有时候会报错为什么？

    C++编译器为了支持函数重载和其他C++特性，会对函数名进行修饰，使得函数名在生成的目标文件中并不是其原始名称。这使得C语言编译器在链接时无法找到这些函数，因为C语言不支持函数重载，也不会对函数名进行修饰。

    `extern "C"`：为了让C++编译器生成与C兼容的函数名，可以使用`extern "C"`来禁用名字修饰。`extern "C"`告诉C++编译器按照C的方式处理指定的代码，使其可以被C代码引用。使用`extern "C"`来保护声明，使得C编译器能够正确地解析和链接这些符号。

24. 静态多态有什么？虚函数原理：虚表是什么时候建立的？为什么要把析构函数设置成虚函数？

    > 静态多态性在编译时确定，主要通过函数重载和模板实现。
    >
    > 1. **函数重载（Function Overloading）**：同一作用域内的多个函数可以有相同的名字，但参数列表不同，编译器根据传递的参数类型和数量选择适当的函数。
    > 2. **模板**：模板允许定义通用函数和类，使得在编译时生成具体类型的实例。

25. map为啥用红黑树不用avl树？（几乎所有面试都问了map和unordered_map区别）

    1. 红黑树：

    应用场景：适用于需要频繁插入、删除、查找操作且数据量较大，要求有序性的情况。
    优点：① 具有平衡性，插入、删除、查找的时间复杂度为O ( l o g n ) O(logn)O(logn)；② 支持有序遍历；③ 可用于实现有序集合、有序映射等场景。
    缺点：①相对于散列表来说，需要更多的内存空间；②对于基于散列的操作，如随机访问，并不擅长。

    2. 散列表：

    应用场景：适用于需要频繁插入、删除、查找操作且数据量较大，不需要有序性的情况。

    优点：①查找、插入、删除的平均时间复杂度为O(1)，最坏时间复杂度是O(n)；②支持随机访问，性能通常比树要好。

    缺点：①可能会出现哈希冲突，需要使用冲突解决方法，如链表法、开放寻址法等；②散列表的元素是无序的，不支持有序遍历。

    - 在空间利用率方面，散列表会占用更多的空间，因为它需要额外的空间来存储哈希表的桶和哈希冲突解决方法所需的链表或探测序列等。而红黑树只需要存储节点数据和指向左右子树的指针。因此，红黑树在空间利用率方面相对于散列表更为优秀。

    - 空间复杂度：散列表的空间复杂度会随着元素数量的增加而增加，而红黑树的空间复杂度与元素数量无关，只与树的高度有关。因此，在元素数量较大时，红黑树的空间复杂度相对于散列表更优秀。

    >1. 红黑树的查询效率比散列表更为稳定，不会因为哈希冲突等因素而导致查询性能下降。在查询方面，红黑树的时间复杂度为O(log n)（n为树中元素个数），而散列表的查询时间复杂度为O(1)。但是，在涉及到大量数据的情况下，红黑树的常数因子更小，查询效率更高。
    >
    >2. 红黑树的插入和删除操作的时间复杂度也为O(log n)，相对于散列表来说更为稳定，不会因为哈希冲突等因素导致性能下降。
    >
    >3. 红黑树是一种平衡二叉搜索树，具有良好的稳定性。在任何情况下，红黑树的平衡性都能得到保持。而散列表的性能会因为哈希冲突等因素而不稳定。
    >
    >4. 红黑树相对于散列表来说，内存消耗更为稳定，不会因为哈希冲突等因素而引起内存的浪费。
    >
    >5. 红黑树是一种有序树结构，可以维护元素的顺序，而散列表是一种无序结构，不适用于需要维护元素顺序的场合。
    >
    >6. 红黑树支持动态扩容和缩容，而散列表需要重新计算哈希函数，重新分配内存等过程，相对来说比较复杂。

26. inline 失效场景

    >`inline` 关键字用于提示编译器尝试将函数的调用展开为函数体，以减少函数调用的开销。然而，`inline` 只是一个建议，编译器可以选择忽略这个建议。

    1. 如果函数体太大或太复杂，编译器可能会选择不进行内联展开，因为这样做可能会导致代码膨胀，进而影响性能。
    2. 递归函数通常不会被内联展开，因为内联展开递归函数会导致无限展开。
    3. 虚函数通常不会被内联展开，因为虚函数的调用在运行时通过虚函数表（vtable）进行，编译器在编译时无法确定实际调用的是哪个函数。

27. C++ 中 struct 和 class 区别

    1. 默认访问权限

    - **struct**：默认情况下，`struct` 的成员和继承是公有的（`public`）。
    - **class**：默认情况下，`class` 的成员和继承是私有的（`private`）。

    2. 继承方式

    - **struct**：`struct` 的默认继承方式是公有继承（`public`）。
    - **class**：`class` 的默认继承方式是私有继承（`private`）。

28. 如何防止一个头文件 include 多次

    1. `#pragma once` 

    2. ```c++
       #ifndef HEADER_FILE_NAME_H
       #define HEADER_FILE_NAME_H
       
       // 头文件内容
       
       #endif // HEADER_FILE_NAME_H
       ```

29. lambda表达式的理解，它可以捕获哪些类型

    > `lambda`表达式允许捕获一定范围内的变量：
    >
    > - `[]`不捕获任何变量
    > - `[&]`引用捕获，捕获外部作用域所有变量，在函数体内当作引用使用
    > - `[=]`值捕获，捕获外部作用域所有变量，在函数内内有个副本使用
    > - `[=, &a]`值捕获外部作用域所有变量，按引用捕获a变量
    > - `[a]`只值捕获a变量，不捕获其它变量
    > - `[this]`捕获当前类中的this指针

30. 友元friend介绍

    在 C++ 中，友元（`friend`）是一种允许一个类或函数访问另一个类的私有（`private`）或受保护（`protected`）成员的机制。友元关系在类定义时声明，并且可以是友元函数或友元类。友元不是类的成员，但它们可以访问类的私有和受保护成员。

    友元的主要用途是**允许紧密耦合的类或函数访问彼此的内部实现，而无需公开其内部数据或方法**。

31. move函数

    用于将一个对象转换为右值引用。右值引用允许移动语义的实现，这是一种通过转移资源（如内存、文件句柄等）而不是复制资源来优化性能的方法。

32. 模版类的作用

33. 模版和泛型的区别

34. 内存管理：C++的new和malloc的区别

    都可以用来在**堆上分配和回收空间**。`new /delete` 是操作符，`malloc/free `是库函数。

    {% note success %}

    **执行 new 实际上执行两个过程**：

    1. 分配未初始化的内存空间（malloc）；

    2. 使用对象的构造函数对空间进行初始化；返回空间的首地址。

    如果在第一步分配空间中出现问题，则抛出` std::bad_alloc `异常，或被某个设定的异常处理函数捕获处理；如果在第二步构造对象时出现异常，则自动调用` delete `释放内存。

    {% endnote %}

    {% note success %}

    **执行 delete 实际上也有两个过程**：

    1. 使用析构函数对对象进行析构；
    2. 回收内存空间（free）。

    以上也可以看出` new `和 `malloc` 的区别，`new `得到的是**经过初始化的空间**，而` malloc `得到的是**未初始化的空间**。所以 `new` 是 `new` 一个类型，而 `malloc `则是`malloc`一个字节长度的空间。`delete `和` free`同理，`delete`不仅释放空间还析构对象，`delete` 一个类型，`free` 一个字节长度的空间。

    {% endnote %}

    **为什么有了 malloc／free ,还需要 new／delete？** 

    因为对于非内部数据类型而言，光用 `malloc／free `无法满足动态对象的要求。对象在创建的同时需要自动执行构造函数，对象在消亡以前要自动执行析构函数。由于 `mallo／free `是库函数而不是运算符，不在编译器控制权限之内，不能够把执行的构造函数和析构函数的任务强加于 `malloc／free`，所以有了` new／delete `操作符。

35. new可以重载吗，可以改写new函数吗

    可以，并添加自定义的打印

36. C++中的map和unordered_map的区别和使用场景

37. 他们是线程安全的吗

    不是线程安全的

38. c++标准库里优先队列是怎么实现的？

39. gcc编译的过程

40. C++ Coroutine

41. extern C有什么作用

    `extern "C"` 是 C++ 中的一个关键字组合，用于指示编译器按照 C 的方式来处理被其修饰的代码。这在需要与 C 代码或使用 C 编译器编译的库进行互操作时特别有用。

    1. 函数名修饰

    C++ 编译器会对函数名进行修饰，以支持函数重载和其他 C++ 特性。这种修饰会导致编译器生成的符号名与源代码中的函数名不同。而 C 编译器不会进行这种修饰。因此，为了确保 C++ 代码可以链接到使用 C 编译器编译的库（或反之），需要禁用 C++ 的函数名修饰，这正是 `extern "C"` 的作用。

    2. C 和 C++ 互操作

    通过 `extern "C"`，可以确保 C++ 编译器生成的符号名与 C 编译器生成的符号名匹配，从而使 C++ 代码能够调用 C 函数，或使 C 函数能够调用 C++ 函数。

42. c++ memoryorder/elf文件格式/中断对于操作系统的作

43. C++的符号表

44. C++的单元测试

# 数据结构算法

![image-20240703100614545](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240703100614545.png)

### 常见问题：链表、排序、二叉树

### 1. 数组和链表区别和优缺点

### 2. 快速排序

### 3. 堆排序是怎么做的

### 4. 冒泡排序

### 5. 二分查找（复杂度）

1. hash表数据很大。rehash的代价很高，怎么办

   1. **增量rehash（Incremental Rehashing）**

   增量rehash是一种将rehash操作分摊到多个插入或删除操作中的技术。这样可以避免一次性进行大量数据迁移所带来的性能问题。

   实现方法：

   - **维护两个哈希表**：一个是当前使用的哈希表，另一个是新哈希表。

   - **逐步迁移**：在每次插入或删除操作时，顺便将少量数据从旧哈希表迁移到新哈希表。

   - **状态标志**：使用标志位指示是否处于 rehash 状态，以及当前 rehash 进度。

     1. **增量 rehash**：将 rehash 操作分摊到多次操作中，减少单次操作的开销。

     2. **提前分配足够大的初始容量**：避免频繁的 rehash。

     3. **使用多个哈希表（Sharding）**：将数据分散到多个哈希表中，降低单个哈希表的负载。

     4. **一致性哈希**：减少节点变更时的数据迁移量，特别适用于分布式系统。

     5. **分布式哈希表**：在分布式系统中，通过分片和节点扩展来处理大量数据。

2. 二叉树前序遍历非递归

3. 链表反转

4. 二叉树输出每一层最右边的节点

5. 千万级数组如何求最大k个数？（用最小堆反之最大堆）千万数
   据范围有限， 0 到 1000 ，有很多重复的，按频率排序怎么处理？

6. 计算二叉树层高。

7. 给一个连续非空子数组，找它乘积最大的（动态规划）

8. 排序算法. 哪些是稳定的，哪些不稳定的

9. 树的深度和高度。一开始分别用了一个层序遍历和一个dfs，然后面试官问能否都在一个dfs里面呢，提示了一下在dfs是否可以传一个参数，然后解决了。

10. 布隆过滤器介绍

11. 为什么不用布隆过滤器

12. 数据结构相关，图的种类，表示方法，图有哪些经典算法+描述算法

13. 求最大的k个数字，解法：优先队列（堆）或者快速排序

14. 一个大数问题，解法：转换为字符串解决，这题没写好，leetcode应该有很多类似的问题

15. hash解决冲突 （ 开放定址法、链地址法、再哈希法、建立公共溢出区 ），四种方式详细的过程、思路

16. 链地址法和再哈希法之间的关联和区别，两者分别适用场景，两者底层的数据结构，关联和区别

17. 链表和数组的底层结构设计、关联、区别、应用场景

18. 死锁的概念，进程调度算法怎么解决死锁

# gdb 调试

### 主要体现在实际工作中常用的命令和技巧

1. 怎么debug，怎么看内存泄漏。

   `Valgrind` 

2. gdb 使用 -> 多线程程序切换到某线程栈帧 -> 如何查看寄存器值

   **启动 GDB 并加载程序**

   ```gdb
   gdb ./your_program
   ```

   **设置断点并运行程序**

   ```gdb
   break main
   (gdb) run
   ```

   **列出所有线程**

   ```gdb
   (gdb) info threads
   ```

   **切换到特定线程**

   ```gdb
   (gdb) thread <thread-id>
   ```

   **查看当前线程的栈帧**

   ```gdb
   (gdb) backtrace
   ```

   **切换到特定栈帧**

   ```gdb
   (gdb) frame <frame-number>
   ```

   **查看寄存器值**

   ```gdb
   (gdb) info registers
   ```

3. 怎么分析C++的core文件

   

4. GDB有哪些命令

5. gcc和g++的区别

6. Linux下程序有问题，如何调试？ （答GDB打开，打上Breakpoint进行调试）

# 设计模式

### 主要体现在工作中常用到的设计模式

### 1. 单例模式实现

### 2. 策略模式实现

### 3. 责任链模式

### 4. 组合模式

### 5. 观察者模式

### 6. 模板方法

# 操作系统

### 1. 线程和进程 的区别、应用场景。

### 2. 多线程中各种锁，读写锁，互斥锁

### 3. 内存池

### 4. 内存管理

### 5. 内存写漏

### 6. 如果频繁进行内存的分配释放会有什么问题吗？

1. 如果频繁分配释放的内存很大（>128k）,怎么处理？
2. 虚拟内存以及堆栈溢出相关的问题，堆栈溢出怎么处理等等。
3. 分段和分页的区别
4. 进程间通信原理和方式
5. fork()读时共享写时拷贝
6. 互斥锁+条件变量
7. 如果非堆内存一直在增长，可能哪个区域的内存出了问题（Java）
8. 堆和栈的区别。什么情况下会往堆里放
9. fork函数返回值是怎么实现的
10. 用户级线程和内核级线程的区别
11. **线程池** 和线程开销
12. 线程切换的到底是什么
13. 线程同步共享怎么实现
14. 互斥同步的方法
15. 信号量和自旋锁的区别
16. 查看磁盘、cpu 占用、内存占用命令

17. linux虚拟地址空间结构/动态库地址无关代码
18. top命令排查高占有率进程/top命令的占用率怎么算的
19. 谈谈进程创建后在Linux中的内存分布？（回答内存四区，虚拟
    地址空间，栈内存堆内存）
20. 在Linux系统下，使用for循环，一直进行new操作，会发生
    heap-overflow吗？如果不会，原因呢？（答应该不会，Linux
    系统可能会对此情况进行处理，面试官追问如果不用C++而用
    Java呢，答Java虚拟机等，胡扯了一些）
21. 死锁的概念，进程调度算法怎么解决死锁
22. 讲讲进程管理

# 网络

## 网络原理

1. tcp和udp的原理、区别、应用场景。
2. 为什么握手是三次而挥手需要四次
3. **TCP** 慢启动，拥塞控制实现
4. **HTTP** 是在OSI模型的哪一层
5. HTTPS用到的是对称加密还是非对称加密？分别体现在哪里？
6. http2和http1的区别
7. http1.0 / 1.1 / 2 / 3
8. get和post区别
9. WebSockt
10. tcp/ip五层模型
11. dns服务器用的是什么协议。
12. ping命令 用的是什么协议。在哪一层。
13. 能详细讲一下有限状态机怎么解析http报文吗
14. 如果解析http请求的时候，用户一次性没传完数据，（如果头部
    都没传完，请求报文长度字段都没传完，怎么办）
15. 路由表说一下
16. 路由表为空怎么找到下一跳
17. 粘包拆包是什么，发生在哪一层

18. TCP在什么情况下会出现大量time_wait，哪个阶段出现
19. TCP 包头字段... 标志位-> 建立连接过程，终止连接过程->
    TIME_WAIT, CLOSE_WAIT 分析，属于哪一方？
20. TCP 建立连接过程 -> SYN + ACK 包能不能拆开来发
21. 讲讲quic/听说过哪些快速重传算法/timewait状态干啥用的
22. 提到了TCP，黏包怎么解决？（固定包头接收，指定内存长度）
23. 查看网络状况（以为是netstate，其实是ping、traceroute，紧张忘记说了）
24. 抓包工具？（wireshark，紧张又给忘了靠）
25. TCP 2MSL说一下，为什么

## 网络编程

1. 为什么要用epoll
2. epoll实现原理，epoll使用的哪种模式， 除了epoll，了解
   select/poll吗
3. 怎么理解多路复用机制的
4. reactor和proactor的好处和坏处。为什么要用reactor而不用
   proactor
5. select怎么用。底层原理
6. select为什么只能支持 1024 个。poll和epoll是怎么解决这个问题
   的。
7. epoll 底层为什么用红黑树不用hash
8. ET和LT的区别、IO多路复用
9. 游戏中数据传输用啥协议（有没有改进的协议，基于UDP的可靠
   传输）
10. 项目架构（webserver）两种高并发模式（问的很细）
11. 除了Reactor模型，还有什么模型

# 数据库



### 2. 数据库的架构

### 3. 不同引擎对 索引 的支持



1. MySQL有哪些引擎

MyISAM、InnoDB、MERGE、MEMORY(HEAP)、BDB(BerkeleyDB)、EXAMPLE、FEDERATED、ARCHIVE、CSV、BLACKHOLE

1. InnoDB和MyISAM的区别

   1. `InnoDB`支持事务，`MyISAM`不支持。对于`InnoDB`每一条`sql`语句都默认封装成事务，自动提交
   2. `InnoDB`支持外键，`MyISAM`不支持。对于一个包含外键的`InnoDB`表转为`MyISAM`会失败
   3. 锁粒度：`InnoDB`支持表、行级锁（默认）支持更高的并发能力，而`MyISAM`只支持表级锁
   4. 存储结构：`InnoDB`表的数据和索引存储在共享表空间或者独立表空间文件中；而`MyISAM`表的数据存储在` .MYD `文件中，索引存储在 `.MYI `文件中
   5. 崩溃恢复：`InnoDB`有较强的崩溃恢复能力，，使用重做日志（Redo Log）和撤销日志（Undo Log）来恢复数据；而`MyISAM`崩溃恢复能力较弱，主要依赖于修复工具（如 `myisamchk`）来修复损坏的表。

2. 隔离级别

   >脏读：一个事务读到了另一个未提交事务修改过的数据（脏读只在读未提交隔离级别才会出现）
   >
   >不可重复读：一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值。（不可重复读在读未提交和读已提交隔离级别都可能会出现）
   >
   >幻读：一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来。（幻读在读未提交、读已提交、可重复读隔离级别都可能会出现）
   >
   >| 名称           | 描述                                                         |
   >| -------------- | ------------------------------------------------------------ |
   >| **脏读**       | 指一个事务读取了另外一个事务未提交的数据。                   |
   >| **不可重复读** | 在一个事务内读取表中的某一行数据，多次读取结果不同。         |
   >| **虚读(幻读)** | 是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。 |
   >
   >**MySQL的默认隔离级别（可重复读）**

   1. 读未提交
   2. 读已提交
   3. 可重复读
   4. 串行化

   >- 读未提交：一个事务读取到其他事务未提交的数据；这种隔离级别下，查询不会加锁，一致性最差，会产生脏读、不可重复读、幻读的问题
   >- 读已提交：一个事务只能读取到其他事务已经提交的数据；该隔离级别避免了脏读问题的产生，但是不可重复读和幻读的问题仍然存在；
   >- 读已提交事务隔离级别是大多数流行数据库的默认事务隔离级别，比如` Oracle`，但是不是` MySQL `的默认隔离界别
   >- 可重复读：事务在执行过程中可以读取到其他事务已提交的新插入的数据，但是不能读取其他事务对数据的修改，也就是说多次读取同一记录的结果相同；该个里级别避免了脏读、不可重复度的问题，但是仍然无法避免幻读的问题
   >- 串行化：事务串行化执行，事务只能一个接着一个地执行，并且在执行过程中完全看不到其他事务对数据所做的更新；缺点是并发能力差，最严格的事务隔离，完全符合ACID原则，但是对性能影响比较大
   >
   >
   >| **事务隔离级别**             | **脏读** | **不可重复读** | **幻读** |
   >| ---------------------------- | -------- | -------------- | -------- |
   >| 读未提交（read-uncommitted） | 是       | 是             | 是       |
   >| 读已提交（read-committed）   | 否       | 是             | 是       |
   >| 可重复读（repeatable-read）  | 否       | 否             | 是       |
   >| 串行化（serializable）       | 否       | 否             | 否       |

3. 最左前缀原则

4. MySQL的集群是用什么样的方式去增加并发量

   主从复制

5. 除了读写分离还有吗？

6. mysql的隔离级别和锁。

7. 数据库delete和trancate区别(这个trancate没用过，没说出来)

   **DELETE**：用于删除表中符合条件的行，可以加 `WHERE` 子句来指定删除的条件。**DELETE**：删除行不会重置表的自动递增计数器

   **TRUNCATE**：用于快速删除表中的所有行，不能使用 `WHERE` 子句。删除所有行后，表的结构和索引仍然保留。**TRUNCATE**：通常会重置表的自动递增计数器

8. mysql索引（B+树）

9. B树和B+树的区别

   - **B+树内节点不存储数据，所有 data 存储在叶节点导致查询时间复杂度固定为 log n。而B-树查询时间复杂度不固定，与 key 在树中的位置有关，最好为O(1)。**
   - **B+树叶节点两两相连可大大增加区间访问性，可使用在范围查询等，而B-树每个节点 key 和 data 在一起，则无法区间查找。**B+树可以很好的利用局部性原理
   - **B+树更适合外部存储。由于内节点无 data 域，每个节点能索引的范围更大更精确**

   | 特性     | B树                            | B+树                             |
   | -------- | ------------------------------ | -------------------------------- |
   | 范围查询 | 效率较低                       | 效率高，叶子节点形成链表         |
   | 节点内容 | 键和值                         | 内部节点只存储键，叶子节点存储值 |
   | 查找路径 | 可能在内部节点或叶子节点找到值 | 总是在叶子节点找到值             |
   | 应用场景 | 频繁插入、删除、查找           | 大量范围查询，高效遍历           |
   | 树的高度 | 相对较低                       | 相对较高，但差异不明显           |

10. B+树树高怎么算？树高为 4 能支持多少数据量

    > **B+树的基本性质**：
    >
    > - 一个 B+树的内部节点最多有 m个子节点。
    > - 一个 B+树的内部节点至少有` m/2`个子节点（除根节点外）。
    > - 所有的叶子节点在同一层级。

    $$
    \text{最大数据量} = L \times m^{h-1}（L为每个节点能存储的数据项数量）
    $$

    

11. 数据库ACID怎么实现

    > - **原子性（Atomicity）**
    >
    > 单个事务，为一个不可分割的最小工作单元，整个事务中的所有操作要么全部 commit 成功，要么全部失败 rollback，对于一个事务来说，不可能只执行其中的一部分 SQL 操作，这就是事务的原子性。
    >
    > - **一致性(Consistency)**
    >
    > 数据库总是从一个一致性的状态转换到另外一个一致性的状态。
    >
    > - **隔离性(Isolation)**
    >
    > 通常来说，一个事务所做的修改在最终提交以前，对其他事务是不可见（隔离）的。避免多个事务并发执行的时候不会互相干扰。
    >
    > - **持久性（Durability）**
    >
    > 一旦事务提交，则其所做的修改就会永久保存到数据库中，之后的其他操作或故障都不会对事务的结果产生影响。

    - 持久性是通过 redo log （重做日志）来保证的；
    - 原子性是通过 undo log（回滚日志） 来保证的；
    - 隔离性是通过 MVCC（多版本并发控制） 或锁机制来保证的；
    - 一致性则是通过持久性+原子性+隔离性来保证；

12. binlog(逻辑备份日志)记录的是什么

    用于记录数据库的所有写操作。它是逻辑备份日志的一种，记录了数据的变化，而不是数据的物理存储状态。Binlog 包含了对数据库进行的所有 DDL（数据定义语言）和 DML（数据操纵语言）语句的记录，并且这些记录可以用于数据恢复、复制和增量备份。

13. mysql的mvcc

    MVCC的基本思想是通过保存数据的多个版本，实现数据的快照读（Snapshot Read）和当前读（Current Read），从而支持不同事务隔离级别下的数据一致性。

    > InnoDB在每行记录的后面增加了两个隐藏的列，用于实现MVCC：
    >
    > 1. **事务ID（trx_id）**：记录插入或更新该行的事务ID。
    > 2. **回滚指针（roll_pointer）**：指向旧版本的指针，通过这个指针可以找到该行的前一个版本。
    >
    > **快照读**是指事务在开始时生成一个数据快照，此后所有的读操作都基于这个快照，而不管其他事务是否提交更改。这种方式保证了读一致性，同时不会阻塞写操作。
    >
    > **当前读**是指读取最新版本的数据，并且会锁住读取到的数据行，避免其他事务修改。当前读一般用于需要获取最新数据并可能进一步修改的场景。

14. mysql锁，每个锁的应用场景

    1. 表级锁：表级锁是对整张表进行加锁，它可以分为读锁和写锁两种类型。
       1. 读锁（共享锁）：读锁也称为共享锁，它可以允许多个用户同时获取读锁并读取相同的数据，但是禁止写入。读锁之间是不互斥的，也就是说多个用户可以同时获取读锁。
       2. 写锁（排他锁）：写锁也称为排他锁，它在获取锁期间，其他用户无法获取读锁或写锁。也就是说，在一个事务获取写锁期间，其他用户无法读取或写入相同的数据。
    2. 行级锁：行级锁是对表中的某一行或某几行数据进行加锁，它可以更细粒度地控制并发访问。行级锁主要是通过**索引**来实现的。
       1. 读锁：共享锁也称为读锁，它允许其他事务获取相同数据的读锁，但是禁止其他事务获取写锁。
       2. 写锁：排他锁也称为写锁，它在获取锁期间，其他事务无法获取读锁或写锁。也就是说，在一个事务获取写锁期间，其他事务无法获取相同数据的读锁或写锁。

    > 1. 并发读写
    >
    > 2. 防止脏读

    

15. 什么情况下会照成死锁，举个例子

    > 假设我们有两个事务 T1 和 T2，它们分别需要访问两张表 A 和 B。具体步骤如下：
    >
    > **事务 T1**
    >
    > 1. 锁住表 A
    > 2. 尝试锁住表 B
    >
    > **事务 T2**
    >
    > 1. 锁住表 B
    > 2. 尝试锁住表 A
    >
    > 如果这两个事务按以下顺序执行，则会发生死锁：
    >
    > 1. 事务 T1 锁住表 A。
    > 2. 事务 T2 锁住表 B。
    > 3. 事务 T1 尝试锁住表 B，但表 B 已被 T2 锁住，因此 T1 进入等待状态。
    > 4. 事务 T2 尝试锁住表 A，但表 A 已被 T1 锁住，因此 T2 也进入等待状态。
    >
    > 此时，T1 和 T2 都在等待对方释放锁，从而导致死锁。

16. **事务** 安全（隔离级别）

17. 你的项目死锁怎么检测的

    **死锁检测**：数据库系统可以定期检查是否存在死锁，并在检测到死锁时选择一个事务作为“牺牲者”来回滚，从而解除死锁。

    **死锁预防**：通过限制事务的资源申请顺序或限制事务持有资源的时间来预防死锁的发生。

    **死锁避免**：事务在申请资源之前，预先判断这样做是否会导致死锁，如果可能会导致死锁，则该申请被拒绝。

    预防死锁的策略

    1. **加锁顺序**：所有事务按照相同的顺序请求资源，可以有效避免环路等待条件。
    2. **锁定粒度**：使用较小粒度的锁（如行锁而不是表锁）可以减少锁冲突的可能性。
    3. **超时机制**：为事务设置锁等待时间，如果超过该时间则自动回滚事务。

18. 数据库三大范式（忘了）

19. 如何加快数据检索的效率

20. .注册登陆的用户名和密码存在哪里？（数据库）

21. 面试官灵魂 4 连问：乐观锁与悲观锁的概念、实现方式、场景、优缺点？

22. 哪几种常见的 signal? SIGSEGV... -> 正常终止程序的信号？->kill 进程，几号信号？

    > 在Linux系统中，信号（signal）是一种用于进程间通信的机制。信号可以通知进程发生了某个事件，如非法内存访问、除零错误、用户请求终止进程等。
    >
    > 常见信号
    >
    > **SIGSEGV (11)**：段错误信号，指示进程试图访问未分配的内存（非法内存访问）。
    >
    > **SIGINT (2)**：中断信号，通常是用户按下 `Ctrl+C` 组合键时发送的信号，用于请求终止进程。
    >
    > **SIGKILL (9)**：杀死信号，强制终止进程，无法被捕获或忽略。
    >
    > **SIGTERM (15)**：终止信号，通常用于请求进程终止，可以被捕获和处理（以便进程进行清理操作）。
    >
    > **SIGHUP (1)**：挂起信号，通常在终端断开时发送给会话首进程，通常用于重新加载配置文件。
    >
    > **SIGQUIT (3)**：退出信号，通常是用户按下 `Ctrl+\` 组合键时发送的信号，用于生成进程核心转储（core dump）。
    >
    > **SIGABRT (6)**：中止信号，通常由调用 `abort` 函数的进程发送，用于生成核心转储。
    >
    > **SIGALRM (14)**：定时信号，时钟定时器到期时发送，用于实现定时操作。
    >
    > **SIGUSR1 (10) 和 SIGUSR2 (12)**：用户自定义信号，供用户定义和使用。
    >
    > **SIGPIPE (13)**：管道破裂信号，进程写入一个无读者的管道时发送。

23. 一千五百万行数据如何快速找到某一行数据，给出方案，设计数据库表结构

24. sql优化

25. 事务

26. 什么情况下使用读已提交

27. 对于脏读的理解

28. 慢查询怎么看，怎么优化

29. 联合索引(a,b,c)，where a, b, c和where b, a, c区别

30. 是否了解db底层

## redis

```
1. redis有什么数据结构
2. 设计一个存储字符串kv对的数据结构，要考虑并发和持久化存储
3. redis 基本数据结构... zset-> zset 底层实现？-> skiplist 和 red-
black tree 对比？
4. 对于redis的理解
5. redis在项目中进行怎么样的使用
6. redis 为什么读取速度那么块 （io、单线程、内存）
7. 为什么redis单线程会快 （完全基于内存、单线程避免不必要的
上下文切换、cpu消耗、加锁问题。。。）
8. 对于很多文件和数据，怎么进行数据的查找、排序，使用什么样的数据结构 （类似于TopK、这个主要是让你进行优化、类似于位图、hash、过滤器之类的）
```

# 组件应用

```
1. 用户认证和鉴权（jwt）
2. 从url下图片 10000 张图片（写了想法，代码没写出来，http的api忘了）， 10 台机器并行下载，怎么实现（主线程给子线程分配下载任务，从线程池取（给自己挖坑））。
3. 雪花算法原理
4. 分布式锁
5. redis和MySQL数据一致性相关设计
6. 长短连接的区别和应用场景
```

### 7. RAII实现数据库连接池，怎么实现的

1. http服务器，他的目标是什么，通过什么方式实现的
2. 负载均衡的一些场景问题
3. 为什么用vector实现缓冲区，有没有想过别的数据结构
4. 小根堆定时器是怎么弄的。如果一次pop一个的话。高并发情况
   下会不会有问题
5. 心跳检测如何实现？
6. 为什么用小根堆实现定时器？还有哪些实现方式？
7. 现场手撕定时器实现
8. 为什么要用多线程。多进程可以吗（webserver的）
9. 为什么要用 **线程池** ，线程池中的线程是怎么运作的?
10. 生产者消费者，信号量的使用
11. 队列空时，消费者和生产者会发生什么
    线程池请求队列是用什么实现的？（链表）
12. .C++多线程并发问题（场景千万级数量级怎么处理）
13. 哪几种常见的 signal? SIGSEGV... -> 正常终止程序的信号？->
    kill 进程，几号信号？
14. 什么情况下会使用静态变量
15. 多线程读写同一个静态变量你是怎么解决的
16. 用过无锁编程吗，知道原子量吗

# 开放性问题

```
1. 介绍自己的c++项目，遇到的难点，实现了那些功能
2. 看过源码嘛，轻量级服务器项目
3. 计算机基础知识是怎么去补滴，之后的技术/职业规划
4. 物联网有个简版的MQ协议叫做MQTT，你可以想一下扫码支付
使用的机器，这些机器的服务器是怎么做到跟这千万级别的客户
端通信的？你扫码支付完之后，服务器是怎么精准返回你这个客
户端说它收到了多少钱？
5. 情景题。手机店。不同品牌的不同型号手机有不同的业务逻辑。
怎么设计系统
```

### 6. 如果有两个服务器，一个服务器坏了，另一个服务器怎么判断并接手坏的服务器的用户数据（共用一个堆）

### 7. 服务器进行过压测么

### 8. 场景设计问题，UDP设计安全可靠的文件传输

### 9. 客户端资源下载到一半突然网络中断怎么办，有进行处理吗？

### 10. 有进行过压力测试吗？

### 11. 在学校里或者公司中最有成就感的事。

### 12. 井盖为什么是圆的