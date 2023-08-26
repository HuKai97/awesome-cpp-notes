[TOC]
# 基础8、可调用对象类型
基础：base7

## 8.1、可调用对象和std::function
《C++ Primer》P511

* 可调用对象：指可以像函数一样被调用的对象

* C++中的可调用对象类型
    * 函数
        ```cpp
        void f1(int x) { std::cout << x << std::endl; }
        int a = 2;
        f1(a);
        ```
    * 函数指针: 函数指针可以退化为函数
        ```cpp
        void (*fPtr)(int) = &f1;
        int a = 3;
        fPtr(a);
        ```
    * lambda表达式
        ```cpp
        int a = 3
        auto f2 = [](int x) { std::cout << x << std::endl;};
        f2(a);
        ```
    * 重载了调用运算符的类
        ```cpp
        class F3 {
        public:
            void operator()(int x) {
                std::cout << x << std::endl;
            }
        };

        int a = 3;
        F3 f3;
        f3(a);
        ```
    * std::bind：是c++11标准库中引入的一个函数，可以以一种灵活的方式来延迟调用函数，并在调用时提供参数。如例子中将函数f4和其参数10绑定成一个可调用对象（即函数对象），绑定时，std::bind会创建一个闭包。再调用这个可调用对象closure，传入参数5，这样函数f4就传入了两个参数10和5。
        ```cpp
        void f4(int x, int y) {
            std::cout << x + y << std::endl;
        }

        // std::placeholder表示函数的第一个参数
        // std::placeholder表示函数的第二个参数
        auto c1 = std::bind(&f4, std::placeholders::_1, 10);
        c1(5);  // 15

        // 等价于
        auto c2 = std::bind(&f4, std::placeholders::_1, std::placeholders::_2, 5);
        c2();
        ```
* std::function：上述5种可调用对象是5种不同的类型，使用常见的STL容器是无法同时封装这五种类型的，所以引入std::function模板。std::function是C++11标准库中的一个类模板，用于封装不同的可调用对象。
    ```cpp
    int Add(int a, int b) {
        return a + b;
    }

    int sub_(int a, int b) {
        return a - b;
    }
    int (*Sub)(int, int) = &sub_;

    auto Mul = [](int a, int b) { return a * b; };

    struct Div{
        int operator()(int a, int b)
        {
            return a / b;
        }
    };

    int mod_(int a, int b) {
        return a % b;
    }
    auto Mod = std::bind(&mod_, std::placeholders::_1, std::placeholders::_2);

    std::map<std::string, std::function<int(int, int)>> funMap = {
            {"+", Add},
            {"-", Sub},
            {"*", Mul},
            {"/", Div()},
            {"%", Mod}};
    ```