[TOC]
# 基础7、lambda表达式（匿名函数）初探
## 7.1、lambda表达式的底层实现原理
《c++ primer》58页
* 总结：lambda表达式的本质是编译器构造出一个匿名类，这个类重载了括号()运算符

* 定义一个lambda匿名函数, 它为什么有大小？
    ```cpp
    size_t sz = 10;
    auto sizeComp = [sz](const string &a) {
        // sz = 21;   // 错误 因为operator是一个const成员函数
        return a.size()>sz;
    };
    auto a = sizeof(sizeComp);   // 8  
    ```
* 其实c++底层编译器会将lambda匿名函数转换为一个可调用的类重载operator运算符，并且将这个类实例化出一个对象（可调用），这个类对象是有大小的
    ```cpp
    class SizeComp{
    public:
        SizeComp(size_t n): sz(n) {}
        // 第一个const: a不能变   第二个const: 类成员sz不能变
        bool operator()(const string &a) const {return a.size()>sz;}
    private:
        size_t sz;  // 如果lambda是按值捕获的 那么这里是个值类型 如果是引用类型 这里也是引用类型
    };
    ```

## 7.2、lambda表达式的基础语法
《现代c++语法核心特性解析》58页
* 形式：[captures] (params) specifiers exception -> return {body}
    * ==[captures] 捕获列表==
        * 只能捕获非静态的局部变量(因为全局变量和静态变量不需要捕获就可以直接使用)，可以是值、引用、或组合使用
            ```cpp
            int c = 2;

            int main(){
                int a = 1;
                static int b = 2;
                int d = 3;
                auto fun1 = [a, &d] {};      // 只能捕获非静态局部变量
                // auto fun2 = [b] {};   // 错
                // auto fun3 = [c] {};   // 错
                return 0;  
            }
            ```
        * 捕获发生在lambda表达式定义的时候(因为在定义lambda表达式时系统就构建了一个可调用类，并创建了一个类对象，所以此时类已经初始化完成了，类成员变量sz的值就已经定好了=10，但是如果是引用的话，类成员变量sz就已经和外部sz绑定在一起了)，而不是使用的时候
            ```cpp
            int a = 10;
            auto fun1 = [a]() {std::cout << a << std::endl;};
            auto fun2 = [&a]() {std::cout << a << std::endl;};
            a = 20;
            fun1();  // 10
            fun2();  // 20
            ```
        * c++14提出更广义的捕获：可以传入右值，既可以防止对象本身不可拷贝，又可以防止无意义的拷贝
            ```cpp
            int a = 10;
            // c++11可以理解为这三处的a都是相同的
            auto fun1 = [a] () { std::cout << a << std::endl; };
            // c++14广义的捕获，认为捕获是下面这种拷贝的形式 
            auto fun2 = [a1 = a] { std::cout << a1 << std::endl; };
            // 当传入一个右值或无法拷贝的值 也可以完成捕获
            auto i = std::make_unique<int>(1);  // 无法拷贝i
            // std::move转成右值 完成捕获
            auto add = [v1 = 1, v2 = std::move(i)](int x, int y) {
                return x + y + v1 + (*v2);
            };
            ```
        * 特殊的捕获方法c++11-14: [=]、[&]、[this], c++17: [*this]
            * [=]：捕获所有局部变量的值，包括this(实际上很智能，用到哪些变量才会捕获哪些变量)
                ```cpp
                int a = 2;
                int b = 3;
                auto f1 = [=]() { std::cout << a << b << std::endl; };
                auto f2 = [=]() { std::cout << a << std::endl; };
                auto c = sizeof(f1);  // 8
                auto d = sizeof(f2);  // 4
                f1();  // 2  3
                f2();  // 2
                ```
            * [&]：捕获所有局部变量的引用，包括this：同上
            * [this]：捕获this指针，可以让我们使用this类型的成员变量和函数, 通常用在类中的lambda表达式
                ```cpp
                class A{
                public:
                    void print() { std::cout << "Class A" << std::endl;}
                    void test(){
                        auto f = [this] {
                            print();
                            x = 5;
                            std::cout << x << std::endl;
                        };
                        f();
                    }
                private:
                    int x;
                };

                A a;
                a.test();   // Class A  5
                ```
            * [*this]：捕获this指向的对象的副本：多线程时更安全
    * ==params 可选参数列表==: 函数列表，c++11和正常函数一样使用，但是c++14开始可以使用auto, 再根据函数调用推导出auto的类型;
        ```cpp
        auto foo = [] (auto a) {return a;};  
        int a = foo(3);  // 推导出auto=int
        ```
    * ==specifiers 可选限定符==：默认是const，说明这个函数不能对捕获列表里的值进行修改，相当于类中的const成员函数，如果需要修改可以加上mutable；
        ```cpp
        size_t sz = 10;
        auto sizeComp = [sz](const string &a) mutable{
            sz = 21;
            return a.size()>sz;
        };
        ```
    * ==exception 可选异常说明符==：可以使用noexcept指明函数是否会抛出异常；
    * ==->ret 可选返回值类型==: 通常可以省略，大多数可以直接在lambda前面添加auto自行推导，但是初始化列表不行；
    * ==body lambda函数函数体==：和正常的函数体一模一样；

## 7.3、lambda表达式存在的意义
* 直观理解：lambda表达式就是为了不用给函数起名吗?当然不是，一般lambda表达式也是起名了的；
  
* 前提知识
    * 闭包：是一个函数，它可以访问并操作其创建时所处的上下文中的变量，即使在创建它的作用域中已经不存在时也可以
    * 闭包实现的方式
        * 定义一个类，并在类中重载operator()运算符
        * lambda表达式
        * 函数中定义函数
        * std::bind
* lambda表达式存在的意义
    * 实现匿名函数，无需显示定义具名函数，代码简洁
    * 捕获外部变量，在lambda表达式内部可以使用和操作捕获的外部变量，甚至可以延迟变量的生命周期
    * 创建闭包，lambda表达式本身就是一个闭包，包含了函数定义和捕获的外部变量