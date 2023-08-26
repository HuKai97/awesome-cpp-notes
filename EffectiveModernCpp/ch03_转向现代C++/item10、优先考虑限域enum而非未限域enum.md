[TOC]
# 条款10、优先考虑限域enum而非未限域enum


* 定义：enum表示枚举，用于定义一组有限的命名常量。可以提高代码的可读性和可维护性。
* 示例
    ```cpp
    // 定义
    enum Color {
        red,
        green,
        blue
    };
    // 使用
    Color coror = red;
    ```
* enum本质：enum中的每个元素的底层都是一个由编译器决定的整型，而且这个整型可以手动设置，甚至可以是相同的
    ```cpp
    // 三个元素的类型都是Color 但是底层类型是一个int
    enum Color {
        red,    // 0
        green,  // 1
        blue = 5    // 2
    };
    ```
* 未限定enum的问题  
    * 作用域是全局的，所以两个enum中的元素不能重名
        ```cpp
        enum A {
            a,b 
        };

        enum B {
            a  // 报错 重名
        };
        ```
    * 可以隐式转换为整型,但是int无法隐式转换为enum，而且显示转换时会发生unkown
        ```cpp
        enum School {
            teacher,
            student
        };
        enum Company {
            manager,
            employee
        };

        School a = student;
        int b = student;

        // School c = 10;
        School d = static_cast<School>(10);  // unknown:10  10即不是student也不是teacher
        ```
    * 通常情况下无法前置声明，因为不知道要分配多大空间，除非在声明与定义处都指定其整型的类型
        ```cpp
        enum School;  // 前置声明报错 不知道给School分配多大的内存
        class A{
        private:
            School _enumSchool;  
        };
        enum School {
            teacher,
            student
        };
        ```
* 限域enum: 在enum后面加一个class，限域enum可以完美解决限域enum的三个问题
    * 实例
        ```cpp
        // 定义
        enum class Number;  // 可以前置声明，默认Number是int类型，按int类型计算所占的大小
        class A{
            Number a;
        };
        enum class Number {
            a,
            b
        };

        // 用法
        // Number n1 = a;  // 错 元素范围限定在enum内部
        Number n1 = Number::a;

        // int n2 = Number::a;  // 错 无法隐式转换为整型
        int n2 = static_cast<int>(Number::a);  // 可以显示转换
        ```
* c++17新增特性：可以使用列表初始化有底层类型的枚举对象（限域或不限域都可以）
    ```cpp
    enum class Number {
        a,
        b
    };

    Number n1 = Number::a;  // c++14
    Number n2{0};           // c++17
    Number n3{Number::b};
    ```
* c++20新增特性：使用using打开限域enum
    ```cpp
    enum class Color {
        Red,
        Green,
        Blue
    };
    void print(Color c){
        switch(c) {
            using enum Color;  // 打开限域enum Color 作用域在switch之内 直接使用Color中元素
            case Red:
                std::cout << "red" << std::endl;
                break;
            case Green:
                std::cout << "green" << std::endl;
                break;
            case Blue:
                std::cout << "blue" << std::endl;
                break;
            default:
                break;
        }
    }
    ```
