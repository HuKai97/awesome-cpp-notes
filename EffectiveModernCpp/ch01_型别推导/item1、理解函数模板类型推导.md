[TOC]
# 条款1、理解函数模板类型推导
基础: base1、base2、base3

## 1.1、前提知识
1. 模板中的const写在T前和T后是一样的，特殊的当T推导为int *，此时const都是顶层const，都是修饰param;
    ```cpp
    // 下面两种写法是一致的
    template <typename T>
    void f(const T param);  // 1
    void f(T const param);  // 2
    int a = 10;
    f(&a);  // 此时T=int* const是顶层const  1和2都可以被调用
    ```
2. 顶层const不构成重载
    ```cpp
    // 两个函数都会被解析成void f(int a)  不构成重载
    void f(int a);
    void f(const int a);   // 顶层const 
    ```
3. 指针的引用：是一个引用 引用的对象是一个指针
    ```cpp
    int a = 10;
    int *b = &a;
    int *&c = b;
    ```
4. 鬼畜的函数指针和函数引用
    * 函数指针的底层const似乎只能用类型别名表示出来
        ```cpp
        int f1(int);
        int (*f2)(int) = f1;
        int (*const f3)(int) = f1;  // 函数的顶层const
        // const int (*f4)(int) = f1;  // 错 这种写法只能说明函数返回值是const int 而不是函数底层const
        // int (const *f5)(int) = f1;   // 错 
        using F = int(int);
        const F *ff1 = f2;   // 函数的底层const
        F *const ff2 = f2;   // 函数的顶层const
        ```
    * 函数引用的底层const会被编译器忽略
        ```cpp
        int f1(int);
        int (&f2)(int) = f1;
        using F = int(int);
        const F &ff1 = f1;  // 函数引用的底层const被忽略了 -> int(int)
        ```
5. 万能引用和引用折叠
    * 万能引用：形式只能是T&&（const T&&不是万能引用），可以根据实参自动推断T是T还是T&,万能引用只出现在模板中；
    * 引用折叠：T& &、T& &&、T&& &可以折叠为T&；T&& &&可以折叠为T&&

## 1.2、函数模板类型推导
* 函数模板一般形式
    ```cpp
    // 定义
    template<typename T>        // 固定写法
    void f(ParamType param);    // 分情况讨论
    // 调用
    f(expr);                    // 分情况讨论
    ```
    * ParamType的可能情况(3种重复)->7种情况
        1. T=const T、T*=T* const、T&、const T*=const T* const, const T&;
        2. T&&、const T&&;
    * expr的可能的18种情况(3种重复)->15种情况
        1. int=const int、int*=int* const、int&、const int*=const int* const、const int&; string字面量+其他字面量;
        2. int&&、const int&&;
        3. int[10]、int(*)[10]、int(&)[10]、bool(int,int)、bool(*)(int,int)、bool(&)(int,int);
    * 理解诀窍
        * 1理解顶层const和底层const，缺什么就补什么;
        * 2理解万能引用
        * 3理解数组和函数都可以退化为指针;
* 分情况讨论
    1. ParamType=T或const T，param和expr没有什么限制
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(T param){
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // T=int ParamType=int
        int *aptr = &a;
        f(aptr);   // T=int* ParamType=int*
        int &aref = a;
        f(aref);   // T=int ParamType=int

        const int ca = 20;   // 等价于int a; 顶层const被忽略
        const int *captr = &ca;  // 顶层const+& -> 底层const
        f(captr);  // T=const int*  ParamType=const int*
        const int &caref = ca;  // 引用不遵循这条准则 底层const也没关系
        f(caref);  // T=int  ParamType=int  本质上传入的就是ca -> int
        int *const acptr = &a;   // 等价于int *aptr
        const int *const cacptr = &a;  // 等价于 const int *captr

        f(10);  // T=int  ParamType=int
        f("hello world");  // T=const char*  ParamType=const char*  const char[12]数组退化为指针const char*

        // 2
        int array[2] = {0, 1};
        f(array);  // T=int*  ParamType=int*  数组退化为指针
        int(*arrayptr)[2] = &array;  
        f(arrayptr);  // T=int(*)[2]  ParamType=int(*)[2]  一个指针 指向int[2]数组
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int*  ParamType=int*  引用和原始一样 数组退化为指针

        func;
        f(func);   // T=void (*)(int,int)  ParamType=void (*)(int,int)  函数退化为函数指针
        void (*funcptr)(int, int) = func;
        f(funcptr);  // T=void (*)(int,int)  ParamType=void (*)(int,int)
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (*)(int,int)  ParamType=void (*)(int,int)  引用和原始的一样 退化为指针

        // 3
        int &&arref = std::move(a);  
        f(arref);  // T=int  ParamType=int  右值引用和左值引用一样
        const int &&carred = std::move(ca);  
        f(carred);  // T=int  ParamType=int
        ```
    2. ParamType=T* 或 T* const, 限定了param必须是指针
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(T* param){
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // 错 param必须是指针 不能传入一个int
        int *aptr = &a;
        f(aptr);   // T=int ParamType=int*
        int &aref = a;
        f(aref);   // 错 param必须是指针 不能传入一个int&

        const int ca = 20;   // 等价于int a; 顶层const被忽略
        const int *captr = &ca;  // 顶层const+& -> 底层const
        f(captr);  // T=const int  ParamType=const int*
        const int &caref = ca;  // 引用不遵循这条准则 底层const也没关系
        f(caref);  // 错 param必须是指针 不能传入一个int 本质传入的还是ca
        int *const acptr = &a;   // 等价于int *aptr
        const int *const cacptr = &a;  // 等价于 const int *captr

        f(10);  // 错
        f("hello world");  // T=const char  ParamType=const char*  const char[12]数组退化为指针const char*

        // 2
        int array[2] = {0, 1};
        f(array);  // T=int  ParamType=int*  数组退化为指针
        int(*arrayptr)[2] = &array;  
        f(arrayptr);  // T=int[2]  ParamType=int(*)[2]  一个指针 指向int[2]数组
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int  ParamType=int*  引用和原始一样 数组退化为指针

        func;
        f(func);   // T=void (int,int)  ParamType=void (*)(int,int)  函数退化为函数指针
        void (*funcptr)(int, int) = func;
        f(funcptr);  // T=void (int,int)  ParamType=void (*)(int,int)
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (int,int)  ParamType=void (*)(int,int)  引用和原始的一样 退化为指针

        // 3
        int &&arref = std::move(a);  
        f(arref);  // 错  右值引用和左值引用一样
        const int &&carred = std::move(ca);  
        f(carred);  // 错
        ```
    3. ParamType=const T* 或 const T* const
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(const T* param){    // 底层const
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // 错 param必须是指针 不能传入一个int
        int *aptr = &a;
        f(aptr);   // T=int  ParamType=const int*    const T*=int*  
        int &aref = a;
        f(aref);   // 错 param必须是指针 不能传入一个int& 引用本质上和a一致

        const int ca = 20;   // 等价于int a; 顶层const被忽略
        const int *captr = &ca;  // 顶层const+& -> 底层const
        f(captr);  // T=int  ParamType=const int*    const T*=const int*  底层const
        const int &caref = ca;  // 引用不遵循这条准则 底层const也没关系
        f(caref);  // 错 param必须是指针 不能传入一个int 本质传入的还是ca
        int *const acptr = &a;   // 等价于int *aptr
        const int *const cacptr = &a;  // 等价于 const int *captr

        f(10);  // 错  
        f("hello world");  // T=char  ParamType=const char*  const char[12]数组退化为指针const char*

        // 2
        int array[2] = {0, 1};
        f(array);  // T=int  ParamType=const int*  数组退化为指针  const T*=int*
        int(*arrayptr)[2] = &array;  // const T*=int(*)[2]
        f(arrayptr);  // T=int[2]  ParamType=const int(*)[2]  一个指针 指向int[2]数组
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int  ParamType=const int*  引用和原始一样 数组退化为指针

        func;
        f(func);   // 错  函数指针的底层const只能用类型别名推导  这里推不出来T的类型 报错
        void (*funcptr)(int, int) = func;
        f(funcptr);  // 错 同理
        void (&funcref)(int, int) = func;  
        f(funcref);  // 错 同理

        // 3
        int &&arref = std::move(a);  
        f(arref);  // 错  右值引用和左值引用一样
        const int &&carred = std::move(ca);  
        f(carred);  // 错
        ```
    4. ParamType=T&，必须传入一个左值；因为是引用所以不执行准则，所有的顶层和底层const都不可以忽略；数组和函数引用都不会发生退化;
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(T& param){
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // T=int ParamType=int&
        int *aptr = &a;
        f(aptr);   // T=int* ParamType=int*&
        int &aref = a;
        f(aref);   // T=int ParamType=int&  引用本质传入的还是a

        const int ca = 20;   // 此处 T& param=const int; 引用不是赋值 所以这个顶层const不能忽略
        f(ca);  // T=const int  ParamType=const int&
        const int *captr = &ca;  // 顶层const+& -> 底层const
        f(captr);  // T=const int*  ParamType=const int*&
        const int &caref = ca;  // 引用和const int ca一样
        f(caref);  // T=const int  ParamType=const int&
        int *const acptr = &a;   // 此处T& param=int *const; 引用不是赋值 顶层const不能忽略
        f(acptr);  // T=int *const  ParamType=int *const &
        const int *const cacptr = &a;  // 此处T& param=const int *const; 引用不是赋值 顶层const和底层const都不能忽略
        f(cacptr); // T=const int *const  ParamType=const int *const &

        f(10);  // 错 左值引用传入右值会报错
        f("hello world");  // T=const char[12]  ParamType=const char (&)[12]  数组的引用不会退化为指针 

        // 2
        int array[2] = {0, 1};
        f(array);  // T=int[2]  ParamType=int(&)[2]  数组的引用不会发生退化
        int(*arrayptr)[2] = &array;  
        f(arrayptr);  // T=int(*)[2]  ParamType=int(*&)[2]
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int[2]  ParamType=int(&)[2]  引用和原始一样 

        func;
        f(func);   // T=void (int,int)  ParamType=void (&)(int,int)  函数引用不会发生退化
        void (*funcptr)(int, int) = func;
        f(funcptr);  // T=void (*)(int,int)  ParamType=void (*&)(int,int)
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (int,int)  ParamType=void (&)(int,int)  引用和原始的一样 不会发生退化

        // 3
        int &&arref = std::move(a);  
        f(arref);  // 错  右值引用和左值引用一样
        const int &&carred = std::move(ca);  
        f(carred);  // 错
        ```
    5. ParamType=T&&，即万能引用，传入左值时，ParamType为左值引用，传入右值时，ParamType为右值引用；
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(T&& param){    // 万能引用
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // T=int&  ParamType=int&   T&&=int  
        int *aptr = &a;
        f(aptr);   // T=int*& ParamType=int*&   T&&=int*
        int &aref = a;
        f(aref);   // T=int& ParamType=int&  引用本质传入的还是a

        const int ca = 20;   // 此处 T&& =const int; 引用不是赋值 所以这个顶层const不能忽略
        f(ca);  // T=const int&  ParamType=const int&  
        const int *captr = &ca;  // 顶层const+& -> 底层const   T&&=const int*
        f(captr);  // T=const int*&  ParamType=const int*&   
        const int &caref = ca;  // 引用和const int ca一样   T&&=const int&
        f(caref);  // T=const int&  ParamType=const int&
        int *const acptr = &a;   // 此处T&&=int *const; 引用不是赋值 顶层const不能忽略
        f(acptr);  // T=int *const&  ParamType=int *const &
        const int *const cacptr = &a;  // 此处T&&=const int *const; 引用不是赋值 顶层const和底层const都不能忽略
        f(cacptr); // T=const int *const&  ParamType=const int *const&

        f(10);  // T=int ParamType=int&&   T&&=int  传入右值 ParamType是右值引用
        f("hello world");  // T=const char (&)[12]  ParamType=const char (&)[12]  数组的引用不会退化为指针   "hello wrold"是一个左值

        // 2
        int array[2] = {0, 1};  // T&&=int[2]
        f(array);  // T=int(&)[2]  ParamType=int(&)[2]  数组的引用不会发生退化
        int(*arrayptr)[2] = &array;  // T&&=int(*)[2]
        f(arrayptr);  // T=int(*&)[2]  ParamType=int(*&)[2]
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int(&)[2]  ParamType=int(&)[2]  引用和原始一样 

        func;   // T&&=void(int, int)
        f(func);   // T=void (&)(int,int)  ParamType=void (&)(int,int)  函数引用不会发生退化
        void (*funcptr)(int, int) = func;
        f(funcptr);  // T=void (*&)(int,int)  ParamType=void (*&)(int,int)
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (&)(int,int)  ParamType=void (&)(int,int)  引用和原始的一样 不会发生退化

        // 3
        int &&arref = std::move(a);  
        f(arref);  // T=int&  ParamType=int&   arref是左值  T&&=int
        const int &&carred = std::move(ca);  
        f(carred);  // T=const int&  ParamType=const int&  T&&=const int
        ```
    6. ParamType=const T&，常量引用右侧可以是任何，所以一般都是成立的;因为是引用所以不执行准则，所有的顶层和底层const都不可以忽略；数组和函数引用都不会发生退化;
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(const T& param){      // 底层const
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // T=int ParamType=const int&     const T&=int
        int *aptr = &a;
        f(aptr);   // T=int* ParamType=int* const&    const T&=int*   因为const修饰的是T
        int &aref = a;
        f(aref);   // T=int ParamType=const int&  引用本质传入的还是a

        const int ca = 20;   // 此处const T& param=const int; 引用不是赋值 所以这个顶层const不能忽略
        f(ca);  // T=int  ParamType=const int&    const T&=const int
        const int *captr = &ca;  // 顶层const+& -> 底层const    
        f(captr);  // T=const int*  ParamType=const int* const&    const T&=const int*  第一个const修饰的是T
        const int &caref = ca;  // 引用和const int ca一样
        f(caref);  // T=int  ParamType=const int&     const T&=const int&
        int *const acptr = &a;   // 此处const T&=int *const; 引用不是赋值 顶层const不能忽略
        f(acptr);  // T=int*  ParamType=int *const &   两个const修饰的都是T 可以省略一个    
        const int *const cacptr = &a;  // 此处const T&=const int *const; 引用不是赋值 顶层const和底层const都不能忽略
        f(cacptr); // T=const int*  ParamType=const int *const&     第一个const和最后一个const都是修饰T

        f(10);  // 常量引用右边可以是任意
        f("hello world");  // T=char[12]  ParamType=const char (&)[12]  数组的引用不会退化为指针  

        // 2
        int array[2] = {0, 1};
        f(array);  // T=int[2]  ParamType=const int(&)[2]  数组的引用不会发生退化
        int(*arrayptr)[2] = &array;  
        // f(arrayptr);  // T=int(*)[2]  ParamType=const int(*&)[2]   const T&=int(*)[2]
        int(&arrayref)[2] = array;
        f(arrayref);  // T=int[2]  ParamType=const int(&)[2]  引用和原始一样 

        func;   // const T&=void (int, int)   函数引用的底层const会被忽略
        f(func);   // T=void (int,int)  ParamType=void (&)(int,int)  函数引用不会发生退化
        void (*funcptr)(int, int) = func;   // const T&=void(*)(int,int)  const修饰的是T
        f(funcptr);  // T=void (*)(int,int)  ParamType=void (* const&)(int,int)
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (int,int)  ParamType=void (&)(int,int)  引用和原始的一样 不会发生退化

        // 3
        int &&arref = std::move(a);  
        f(arref);  // T=int ParamType=const int&  右值引用和左值引用一样
        const int &&carred = std::move(ca);  
        f(carred);  // 同上
        ```
    7. ParamType=const T&&, 右值引用，不再是万能引用,只能传入一个右值
        ```cpp
        void func(int, int) {}
        template<typename T>
        void f(const T&& param){    // 万能引用
            std::cout << param << std::endl;
        }

        // 1
        int a = 10;
        f(a);      // 错 无法传入左值
        int *aptr = &a;
        f(aptr);   // 错 同上
        int &aref = a;
        f(aref);   // 错 同上

        const int ca = 20;   // 此处 T&& =const int; 引用不是赋值 所以这个顶层const不能忽略
        f(ca);  // 错 同上
        const int *captr = &ca;  // 顶层const+& -> 底层const   T&&=const int*
        f(captr);  // 错 同上
        const int &caref = ca;  // 引用和const int ca一样   T&&=const int&
        f(caref);  // 错 同上
        int *const acptr = &a;   // 此处T&&=int *const; 引用不是赋值 顶层const不能忽略
        f(acptr);  // 错 同上
        const int *const cacptr = &a;  // 此处T&&=const int *const; 引用不是赋值 顶层const和底层const都不能忽略
        f(cacptr); // 错 同上

        f(10);  // 对 T=int ParamType=const int&&   const T&&=int 
        f("hello world");  // 错 同上

        // 2
        int array[2] = {0, 1};  // T&&=int[2]
        f(array);  // 错 同上
        int(*arrayptr)[2] = &array;  // T&&=int(*)[2]
        f(arrayptr);  // 错 同上
        int(&arrayref)[2] = array;
        f(arrayref);  // 错 同上

        func;   // const T&&=void(int, int)   函数引用的底层const会被忽略  等价于 T&&=void(int,int)万能引用
        f(func);   // T=void (int,int)  ParamType=void (&&)(int,int)  函数引用不会发生退化
        void (*funcptr)(int, int) = func;
        f(funcptr);  // 错 传入函数指针 是一个左值
        void (&funcref)(int, int) = func;  
        f(funcref);  // T=void (int,int)  ParamType=void (&&)(int,int)  引用和原始的一样 不会发生退化

        // 3
        int &&arref = std::move(a);  
        f(arref);  // 错 同上
        const int &&carred = std::move(ca);  
        f(carred);  // 错 同上
        ```