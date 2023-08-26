[TOC]
# 条款18、对于独占资源使用std::unique_ptr
基础：base13

## 18.1、unique_ptr的基础
主要是c++ primer中的内容
* unique_ptr独享它所指向的对象，同一时刻一个unique_ptr对象只能指向一个对象，当unique_ptr对象被销毁时，它所指向的对象也被销毁;
* 当定义一个unique_ptr对象时，不需要mask_shared函数，需要将其绑定到一个new返回的指针上，且初始化必须采用直接初始化方式;
    ```cpp
    unique_ptr<double> p1;  // 指向一个double的unique_ptr空指针
    unique_ptr<int> p2(new int(42));  // 指向一个值为42的unique_ptr指针
    unique_ptr<int> p3 = new int(42);  // 错 必须是直接初始化方式
    ```
* unique_ptr独有的操作
    ![这是图片](../data/unique_ptr.png "Magic Gardens")
* 一个unique_ptr独有所指的对象，所以unique_ptr不支持普通的拷贝和赋值操作;
    ```cpp
    unique_ptr<string> p1(new string("hello world"));
    unique_ptr<string> p2(p1);   // 错  unique_ptr不支持拷贝
    unique_ptr<string> p3;
    p3 = p2;   // 错 unique_ptr不支持赋值
    ```
* 例外：可以拷贝或赋值一个将要销毁的unique_ptr;
    ```cpp
    unique_ptr<int> clone1(int p){
        return unique_ptr<int>(new int(p));  // 拷贝一个临时变量
    }
    unique_ptr<int> clone2(int p){
        unique_ptr<int> ret(new int(p));  // 拷贝一个局部对象
        return ret;
    }
    ```

* 虽然不能拷贝或赋值unique_ptr，但是可以通过调用release或reset函数将指针的所有权从一个(非const)unique_ptr转移给另一个unique_ptr;
    ```cpp
    unique_ptr<string> p1(new string("hello"));
    // p1.release: p1先放弃对"hello"的控制 再返回指向这个空间的指针
    // 将“hello”所有权从p1转移给p2 并将p1置为空 
    unique_ptr<string> p2(p1.release());
    unique_ptr<string> p3(new string("world"));
    // 将”world“的所有权从p3转移给p2  并释放原先p2所指向的内存“hello”
    p2.reset(p3.release());
    // p2.release();  // 错误 会造成内存泄漏
    auto p = p2.release();  // 正确做法
    delete p;  
    ```
## 18.2、effective modern c++知识
* unique_ptr的内存模型：指针 + 可调用对象std::function

* 默认情况下，unique_ptr大小等同于原始指针(8 bytes)，不会有更大的额外的开销。但是当自定义删除器时，unique_ptr大小可能发生改变，当传入一个函数时，这个可调用对象=指向这个函数指针=8个字节，当传入一个lambda表达式时，lambda表达式是匿名对象（1个字节，优化->0字节），所以如果需要传入的删除器过大，需要慎重使用unique_ptr

* unique_ptr不允许拷贝和赋值(只能拷贝将要销毁的unique_ptr)，但是允许移动，移动后原指针被设为nullptr
    ```cpp
    unique_ptr<string> p1(new string("hello world"));
    unique_ptr<string> p2(p1);   // 错  unique_ptr不支持拷贝
    unique_ptr<string> p3;
    p3 = p2;   // 错 unique_ptr不支持赋值
    unique_ptr<int> clone1(int p){  // 对 拷贝一个临时变量
        return unique_ptr<int>(new int(p));  
    }

     unique_ptr<string> p4 = std::move(p1);  // 对 允许移动
    ```

* 智能指针的一个应用场景：unique_ptr作为继承层次结构中对象的工厂函数返回类型
    * 工厂函数的返回类型必须是指针：为了实现多态
    * 选择智能指针：指针是非常不容易维护的，栈上的内存是传不出来，堆上的内存需要手动释放

    * 情况1：不用智能指针也不new,不好，栈上的内存是传不出来的
        ```cpp
        template <typename... T>
        Base* makeBase_test1(T &&...params) {
            Base *p;
            constexpr int numArgs = sizeof...(params);
            if constexpr(numArgs == 1) {
                Derived1 d1(std::forward<T>(params)...);
                p = &d1;
            }
            if constexpr(numArgs == 2) {
                Derived2 d2(std::forward<T>(params)...);
                p = &d2;
            }
            if constexpr(numArgs == 3) {
                Derived3 d3(std::forward<T>(params)...);
                p = &d3;
            }
            return p;
            // return p 返回的是临时变量的地址 是返回不出去的
        }
        ```
    * 情况2：不用智能指针，new，不好，堆上的内存很难控制,makeBase_test2函数返回的指针需要用户手动释放
        ```cpp
        template <typename... T>
        Base* makeBase_test2(T &&...params) {
            Base *p;
            constexpr int numArgs = sizeof...(params);
            if constexpr(numArgs == 1) {
                Derived1 d1 = new Derived1(std::forward<T>(params)...)
                p = d1;
            }
            if constexpr(numArgs == 2) {
                Derived2 d2 = new Derived2(std::forward<T>(params)...)
                p = d2;
            }
            if constexpr(numArgs == 3) {
                Derived3 d3 = new Derived3(std::forward<T>(params)...)
                p = d3;
            }
            return p;
        }
        ```

    * 情况3: 智能指针和new，智能指针p指向堆上的变量，可以返回出去，且当makeBase_test3函数执行结束后系统会自动收回智能指针
        ```cpp
        template <typename... T>
        Base* makeBase_test3(T &&...params) {
            std::unique_prt<Base> p{nullptr};
            constexpr int numArgs = sizeof...(params);
            if constexpr(numArgs == 1) {
                p.reset(new Derived1(std::forward<T>(params)...));
            }
            if constexpr(numArgs == 2) {
                p.reset(new Derived2(std::forward<T>(params)...));
            }
            if constexpr(numArgs == 3) {
                p.reset(new Derived3(std::forward<T>(params)...));
            }
            return p;
        }
        ```

* 智能指针的自定义删除器（有点难 暂时不太懂）
    * 删除器作用：智能指针可以调用它的析构函数（智能指针是一个类）自行管理该指针所指向内存空间的申请和释放，但是当析构函数无法完成需求时，就需要自己定义 释放智能指针所指向资源的方式，即自定义删除器
    * 删除器对象类型
        1. 函数对象作为自定义删除器（16个字节）
            ```cpp
            void del1(P *p) {
                std::cout << "delete1" << std::endl;
                delete p;
            }
            // 模板 删除类型，删除器类型
            std::unique_ptr<P, void(*)(P*)> uptr1(nullptr, del1);
            ```
        2. lambda表达式作为自定义删除器（8个字节）
            ```cpp
            auto del2 = [](P *p) {
                std::cout << "dekete2" << std::endl;
                delete p;
            }
            std::unique_ptr<P, decltype(del2)> uptr2(nullptr, del2);
            ```
