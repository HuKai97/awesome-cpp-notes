[TOC]
# item17、理解特殊成员函数的生成
基础：base10

## 17.1、特殊成员函数
1. 默认构造函数
2. 析构函数
3. 拷贝构造函数
4. 移动构造函数
5. 拷贝赋值函数
6. 移动赋值函数
    ```cpp
    class A {
    public:
        A() {   // 默认构造函数
            std::cout << "Default constructor func called" << std::endl; 
        }
        ~A() {  // 析构函数
            std::cout << "Destructor func called" << std::endl; 
        }
        A(const A &a) : data1(a.data1), data2(a.data2) {   // 拷贝构造函数
            std::cout << "Copy constructor func called" << std::endl;
        }
        A& operator = (const A &a) {    // 拷贝赋值运算
            std::cout << "Copy Assignment op called" << std::endl;
            if(this != &a) {
                data1 = a.data1;
                data2 = a.data2;
            }
            return *this;
        }
        A(A &&a) noexcept : data1(std::move(a.data1)), data2(std::move(a.data2)) {  // 移动构造函数
            std::cout << "Move constructor func called" << std::endl;
        }
        A& operator = (A &&a) noexcept {   // 移动赋值运算
            std::cout << "Move Assignment op called" << std::endl;  
            if(this != &a) {
                data1 = std::move(a.data1);
                data2 = std::move(a.data2);
            }
            return *this;
        }
    private:
        int data1;
        double data2;
    };

    A a1;                    // 默认构造函数
    A a2(a1);                // 拷贝构造函数
    A a3 = a2;               // 拷贝赋值运算符
    A a4(std::move(a1));     // 移动构造函数
    A a5 = std::move(a1);    // 移动赋值运算符
    ```


## 17.2、三(C++11之前)/五(C++11之后)法则
c++98: 1析构函数、2拷贝构造函数、3拷贝赋值运算
c++11: 98基础上，加入4移动构造函数、5移动赋值运算

1. 法则：自定义上面5个函数中任意一个，其他4个也应该显示声明，而不用默认的
2. 原因
    * 若一个类需要析构函数，则代表其合成析构函数不足以释放类所拥有的资源，其中最典型的就是new了一个指针成员，我们需要自己写析构函数来delete释放掉这个指针，以防止内存泄露;
    * 而当类中出现了指针类型的成员，我们必须还要防止浅拷贝问题，所以这个时候又必须自定义其他5个函数（其他5个函数本质上就是在完成类非静态成员的拷贝/移动操作）;

3. 实际上，为了兼容c++98，定义析构函数、拷贝构造函数和拷贝赋值运算符中的某几个，编译器还是会自动生成其他几个
    * 实例：自定义1析构函数，编译器还是会自动生成2拷贝构造函数和3拷贝赋值运算符

4. c++11的及时止损
    * 声明1/2/3，编译器就肯定不会自动生成4/5
        实例1：如果一个类中有虚函数，那么析构函数最好也自定义为虚析构函数（将析构函数定义为虚函数是为了实现多态的正确行为，确保在删除派生类对象时能够正确调用派生类的析构函数），此时4/5会被阻止
    * 自定义4/5, 编译器就肯定不会生成默认的2/3/4/5，一个类的1析构函数一定是需要有的
        * 实验1：为什么4移动构造和5移动赋值没有=delete
        * 原因：在执行4/5时，系统没有生成默认的4/5，转而执行2/3，但是2/3是delete状态，直接报错，系统没有写delete 4/5，但是同样达到了目的
            ```cpp
            // 验证：将下面代码粘贴到https://godbolt.org/  可以发现编译器帮我们生成了析构函数B::~B()
            // inline B(const B &) /* noexcept */ = delete;                  // 2
            // inline B & operator=(const B &) /* noexcept */ = delete;      // 3
            // inline ~B() noexcept = default;                               // 1
            #include <iostream>
            # include <vector>
            #include <memory>
            using namespace std;

            struct B {
                B() = default;
                B(B &&) {}
                virtual void vfunc() {}
                int data1{40};
                std::string data2{"hhh"};
            };


            int main(){
                std::cout << "hello world" << std::endl;
                B b;
                return 0;
            }
            ```
    * 如果没有定义（系统也没生产默认的）4/5，但是代码需要执行4/5，那么编译器会自动调用拷贝构造/赋值
        ```cpp
        class A {
        public:
            A() {   // 默认构造函数
                std::cout << "Default constructor func called" << std::endl; 
            }
            ~A() {  // 析构函数
                std::cout << "Destructor func called" << std::endl; 
            }
            A(const A &a) : data1(a.data1), data2(a.data2) {   // 拷贝构造函数
                std::cout << "Copy constructor func called" << std::endl;
            }
        private:
            int data1{};
            double data2;
        };

        A a1;                      // Default constructor func called
        A a2(std::move(a1));       // Copy constructor func called
        ```