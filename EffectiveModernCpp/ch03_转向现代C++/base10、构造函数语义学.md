[TOC]
# 基础10、构造函数语义学

## 10.1、编译器是如何完善构造函数的
1. 声明时初始化的非静态成员，编译器负责将其安插进构造函数中进行初始化
    * 尽量在构造函数中使用初始化列表方式，不要使用赋值方式，可以免去一次拷贝
    ```cpp
    struct A {
        A(int param) : a(param) { 
            /* 编译器完成安插初始化
                int a = param;  // 替换
                float b = 20.f;
                std::string c{"hello"};
            */
            b = 50.f;   // 对b重新赋值 
        }

        int a = 10;
        float b = 20.f;
        std::string c{"hello"};
    };
    ```
2. 类中未初始化的非静态类成员变量，编译器负责调用这个非静态类成员的的默认构造函数
    ```cpp
    struct B {
        B() { std::cout << "B()" << std::endl; }
        B(int data) : data1(data) { std::cout << "B(int)" << std::endl; }
        int data1;
    };

    struct C {
        C(int data) : data2(data) { 
            /* 编译器完成对未初始化的非静态类成员，调用它的默认构造函数进行初始化
                B();
            */
            std::cout << "C(int)" << std::endl; 
        }
        B b;
        int data2;
    };

    C c(5);   // B()  C(int)
    ```
3. 基类如果存在默认构造函数，编译器负责将其安插在子类构造函数，如3
4. 如果存在虚表，编译器将在构造函数中负责进行虚表指针的设置
    ```cpp
    // https://godbolt.org/  可以清晰的看到在B()中对虚表指针的设置操作
    #include <iostream>
    #include <string>
    #include <vector>

    struct B {
        B() { std::cout << "B()" << std::endl; }
        B(int data) : data1(data) { std::cout << "B(int)" << std::endl; }
        virtual void vfunc() { std::cout << "B::vfunc()" << std::endl; }
        int data1;
    };

    int main(){
        B b;  
        return 0;
    }
    ```


## 10.2、如果定义的类中没有构造函数
1. 如果编译器要做上述4件事，那么编译器就会合成默认构造函数
    ```cpp
    // https://godbolt.org/  可以很清晰的看到系统为我们合成了默认构造函数B()
    #include <iostream>
    #include <string>
    #include <vector>

    struct B {
        virtual void vfunc() { std::cout << "B::vfunc()" << std::endl; }
        int data1;
    };

    int main(){
        B b;  
        return 0;
    }
    ```
2. 否则，编译器不会合成默认构造函数，或者称其默认构造函数是trivial的
3. 类中只有定义了任何一个构造函数，编译器不会合成默认构造函数
    ```cpp
    struct B {
        B(int data) : data1(data) {}
        virtual void vfunc() { std::cout << "B::vfunc()" << std::endl; }
        int data1;
    };

    B b;  // 报错 编译器没有合成默认构造函数
    ```

## 10.3、基类如果不存在默认构造函数，子类中需要手动初始化
```cpp
struct D {
    // 编译器不会合成默认构造函数
    D(int data) : data4(data) {}
    int data4;
};

struct E: public D {
    // E(int data) : data5(data) {}  // 报错 D不存在默认构造函数
    E(int d, int data): D(d), data5(data) {}
    int data5;
};
```


## 10.4、使用using继承基类的构造函数
《现代c++语言核心特性解析》p119
```cpp
struct F{
    F(int a) : data6(a) {}
    F(int a, int b) : data6(a), data7(b) {}
    F(int a, int b, int c) : data6(a), data7(b), data8(c) {}
    F(int a, int b, int c, int d) : data6(a), data7(b), data8(c), data9(d) {}
    void func() { std::cout << "F::func()" << std::endl; }
    int data6;
    int data7;
    int data8;
    int data9;
};

struct G: public F {
    using F::F;  // G继承了F的构造函数
    void func() { std::cout << "G::func()" << std::endl; }
};

int main(){
    F f1(5);
    F f2(5, 6);
    G g1(5);
    G g2(5, 6);
    return 0;
}
```