[TOC]
# 基础3、数组指针和函数指针(这个知识点用处不大 c++建议用容器代替数组)
1. 数组相关的数据类型
    * 数组本质上是type[n]类型，但是它可以退化为指针类型，即数组名通常是该数组的首地址;
        ```cpp
        int array[3] = {1, 2, 3};  // array的数据类型是int[3]  不是int*
        // 但常常听到这句话：数组的名字就是它的首元素的地址  代码表示如下
        // 本质：数组名int[3]退化为指针int*
        int *p1 = array;    // 指向数组首元素的指针 
        ```
    * 数组指针、指针数组和数组引用
        * 数组指针是指针，指针数组是数组，数组引用是引用;
        * c++中没有引用数组，因为数组内存放的必须是对象，而引用不是对象;
        * 规则：有括号先看括号内，没有括号从右往左看;
            ```cpp
            int a = 1;
            int &b = a;
            int *p2[3] = {&a, &a, &a};  // 指针数组 p2是一个数组，数组中的元素都是int*类型
            int (*p3)[3] = &array;      // 数组指针 p3是一个指针，指向int[3]的数组
            int (&p4)[3] = array;       // 数组引用 p4是一个引用，引用int[3]的内容
            int &p5[3] = {b, b, b};     // 错误 没有引用数组
            cout << *p1 << endl;       // 1
            cout << (*p3)[1] << endl;  // 2
            ```
2. 当数组名作为函数参数传递的时候会发生退化
    * 传入数组类型参数会退化为指针类型，指向首元素的地址;
        ```cpp
        // 下面三种方式等价
        void func1(int *a) {}
        void func2(int a[N]) {}     // int a[N]退化为int*  N可以为任何数 甚至可以不填
        void func3(int a[]) {}
        ```
    * 传入数组指针不会发生退化;
        ```cpp
        // 下面三种方式不等价
        void fun1(int a[]) {}  // 传入一个指向数组首元素的指针
        void fun2(int (*a)[5]) {}  // 传入一个指向数组a[5]的指针
        void fun3(int (*a)[100]) {}  // 传入一个指向数组a[100]的指针
        ```
    * 参数是数组引用，也不会发生退化;
        ```cpp
        void fun2(int(&a)[2]);
        int b[2]= {1, 2};
        fun2(b);
        ```
    * 字符串常量在赋值过程中也会发生退化的情况;
        ```cpp
        // "hello world"原本是const char[12] 退化为const char* 
        // 这里的const不能省略, 顶层const在退化之后就变成了底层const
        const char *p = "hello world";
        ```

3. 函数相关的数据类型
    * 函数和函数指针的数据类型
        ```cpp
        bool fun(int a, int b);     // 函数本身 数据类型 = bool (int, int);
        // 定义一个指针，指针指向一个函数，函数的返回类型是bool，函数有两个形参(int, int)
        bool (*funP)(int a, int b);  // 函数指针 数据类型 = bool (*)(int, int);
        ```
    * 函数指针的常见用法
        * 函数指针通常和函数等价(函数本身会退化为一个函数指针)
            ```cpp
            funP = &fun;
            funP = fun;
            ```
        * 函数指针的使用(函数指针通常和函数等价)
            ```cpp
            bool a = fun(1, 2);  
            bool b = (*funP)(1, 2);
            ```
        * 函数指针还可以作为另一个函数的形参(函数指针通常和函数等价)
            ```cpp
            void fun2(int c, bool fun(int, int));
            void fun2(int c, bool (*funP)(int, int));
            // 看起来太繁琐 可以使用类型别名的方式
            using Fun = bool (int, int);
            using FunP = bool (*)(int, int);
            void fun2(int c, Fun fun);
            void fun2(int c, FunP funp);
            ```
        * 函数的返回值只能是函数指针，不能是函数本身
            ```cpp
            // fun(3)是一个函数，参数是int c，返回值是一个指针 指向一个函数，返回值是bool 参数是(int, int)  太反人类了...
            bool (*fun3(int c))(int, int);  
            funP fun3(int c);  // 类型别名写法
            auto fun3(int c) -> bool (*)(int, int); // 也可以使用尾置返回类型
            string::size_type fun2(int c, const int c);
            // 定义fun3函数的返回类型是指向fun2的指针类型
            decltype(fun2) *fun3(const string*);
            ```
        * 函数的引用,函数引用不会发生退化
            ```cpp
            bool (&func2)(int, int) = func1;
            ```