[TOC]
# 条款8、指针初始化时优先考虑nullptr而非0和NULL

* nullptr、0、NULL都是什么类型？
    * 0: int
    * NULL: long long
    * nullptr: std::nullptr_t
        ```cpp
        auto a = 0;         // int
        auto b = NULL;      // long long
        auto c = nullptr;   // std::nullptr_t
        ```

* 正确调用指针版本的函数重载
    * 0、NULL、nullptr都可以初始化指针，但是nullptr是最好的，因为0和NULL都需要进行一次强制类型转换
        ```cpp
        void f(int) { std::cout << "f(int)" << std::endl; }
        void f(bool) { std::cout << "f(bool)" << std::endl; }
        void f(void*) { std::cout << "f(void*)" << std::endl; }

        f(0);   // f(int)
        f(NULL);  // 可以调用3个 所以编译不通过
        f(nullptr);  // f(void*)
        ```


* 0可以初始化指针，但是0赋值给一个变量，再用这个变量初始化指针就会报错，特别注意模板推导时不能混用
    ```cpp
    int a = 0;
    int *b = a;  // 错

    int f1(std::shared_ptr<int> spi) { return 5; }
    double f2(std::unique_ptr<int> upi) { return 5.; }
    bool f3(int *pi) { return true; }
    template<typename FuncType, typename PtrType>
    decltype(auto) func(FuncType f, PtrType ptr) { return f(ptr); }

    // bool func<bool (*)(int *pi), std::nullptr_t>(bool (*f)(int *pi), std::nullptr_t ptr)
    auto res = func(f3, nullptr);   // 调用f3(nullptr)
    // bool func<bool (*)(int *pi), int>(bool (*f)(int *pi), int ptr)
    // 这里0为什么不能初始化pi指针？  int *a = 0不是对的吗？
    auto res = func(f3, 0);   // 错 调用f3(0)  相当于int tmp=0; int*a=tmp; 错
    // 调用智能指针时同理
    auto res = func(f1, 0);   // 错
    auto res = func(f2, 0);   // 错
    ```

