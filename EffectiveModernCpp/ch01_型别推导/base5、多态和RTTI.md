[TOC]
# 基础5、多态和RTTI
基础：base4


## 5.1、多态
* 定义：指的是同一个方法或操作可以在不同的对象上产生不同的行为

* 可分为静态多态（编译时多态）和动态多态(运行时多态)

### 5.1.1、静态多态
* 定义：指在编译时期根据参数的类型和数量来确定调用哪个方法，通常有函数重载、泛型编程和奇异递归模板模式（CRTP）;
    * 实例1：函数重载
        ```cpp
        void print(int a);
        void print(int a, int b);
        void print(float a, float b);
        print(1, 2);
        ```
    * 实例2：泛型编程，模板允许你编写可以适用于不同类型的通用代码，编译器会在编译时为每种具体类型生成必要的代码。这被称为编译时多态。
        ```cpp
        #include <iostream>
        template<typename T>
        T add(T a, T b) {
            return a + b;
        }
        int result1 = add<int>(1, 2); // 调用模板函数add<int>，返回3
        double result2 = add<double>(1.5, 2.5); // 调用模板函数add<double>，返回4.0
        ```
    * 实例3：base9
    
### 5.1.2、动态多态/运行时多态
* 虚函数：虚函数的c++的一个特性，目的是为了实现多态性。在基类中定义一个函数为虚函数，并在派生类中通过virtual关键字重写该函数。当基类指针指向派生类对象，即发生运行时多态时，虚函数的调用是根据实际对象的类型来确定的，而不是根据指针或引用的的类型。
* 定义: 指在运行时期根据对象的实际类型来确定调用哪个方法, 通常需要结合虚函数来实现动态绑定。
    * 实例：基类指针指向派生类对象
        ```cpp
        class A {
        public:
            A() {
                data1_ = new int();
                data2_ = new int();
            }
            virtual void vAfunc1() { std::cout << "A::vAfunc1()" << std::endl; }
            virtual void vAfunc2() { std::cout << "A::vAfunc2()" << std::endl; }

            void func1() { std::cout << "A::func1()" << std::endl; }
            void func2() { std::cout << "A::func2()" << std::endl; }

            virtual ~A() {
                std::cout << "A::~A()" << std::endl;
                delete data1_;
                delete data2_;
            }
        private:
            int *data1_;
            int *data2_;
        };

        class B: public A{
        public:
            B() {
                // 执行子类的构造/析构函数之前会先自动执行基类的构造/析构函数
                data3_ = new int; 
            }
            virtual void vAfunc1() override { std::cout << "B::vAfunc1()" << std::endl; }
            virtual void vBfunc1() { std::cout << "B::vBfunc1()" << std::endl; }
            virtual void vBfunc2() { std::cout << "B::vBfunc2()" << std::endl; }

            void func2() { std::cout << "B::func2()" << std::endl; }
            
            virtual ~B() {
                std::cout << "B::~B()" << std::endl;
                delete data3_;
            }
        private:
            int *data3_;
        };

        A aa = b;         // 不是多态 
        aa.vAfunc1();     // A::vAfunc1()  
        aa.func1();       // A::func1()
        A *aptr = &b;     // 多态 
        aptr->vAfunc1();  // B::vAfunc1() aptr调用的是派生类的虚函数
        aptr->func1();    // A::func1()
        ```
    * 注意事项
        * A aa = b,不是多态，等价于A a = static_cast< A>(b)，转换后的虚表变了；A *aptr = &b是多态，等价于A *a = reinterpret_cast< A*>(&b)，虚表不变，aptr指向b的地址 虚表还是b的虚表 但是找不到data3_;
        * 非虚函数是不存在多态的, 只有调用虚函数才会产生多态;
        * 作为继承关系中的基类，如果需要定义一个析构函数，那么通常需要定义为虚析构函数：为了确保在通过基类指针删除派生类对象时，能够正确调用派生类的析构函数,防止内存泄漏。因为如果析构函数不是虚函数，那么~A和~B就不在虚表中，普通的函数是没有多态的，所以delete aptr只会执行A的析构函数而不会执行B的析构函数，造成内存泄漏。
            ```cpp
            A *aptr = new B();
            delete aptr;  // B::~B()  A::~A()

            // 都不是虚析构函数：A::~A()
            ```
## 5.2、RTTI（Runtime Type Identification, 运行时类型判别）
* 定义：由于多态的存在，一个函数的传入参数如果是一个基类指针或引用，有时我们需要在函数中确认获得的实参对象的类型。主要由typeid和reinterpret_cast两个运算符实现。
    ```cpp
    void test(A *a) {
        if(typeid(*a) == typeid(A)){
            // 相应处理
            A *pa = reinterpret_cast<A *>(a);
            pa->func1();
            std::cout << "A" << std::endl;
        }
        if(typeid(*a) == typeid(B)){
            // 相应处理
            B *pb = reinterpret_cast<B *>(a);
            pb->func1();
            std::cout << "B" << std::endl;
        }
    }

    A a;
    B b;
    test(&a);  // A
    test(&b);  // B
    ```
### 5.2.1、typeid运算符
* 定义：typeid运算符可以用于获取一个表达式的类型信息，返回一个std::type_info类型的对象，该对象包含了表达式的类型信息（返回类虚表中的type_info的信息）
* 作用：当作用于类类型且含有虚函数时，返回指针或引用所指对象的动态类型；当作用于非类类型或者是一个不包含虚函数的类类型，返回返回指针或引用所指对象的静态类型
* 形式：typeid(e)，e是任意表达式或类型名字
* 实例
    ```cpp
    Derived *dp = new Derived;
    Base *bp = dp;
    // typeid作用于指针所指的对象时 返回指针所指对象的动态类型
    if (typeid(*bp) == typeid(*dp))  // true
        cout << "equal" << endl;
    if (typeid(*bp) == typeid(Derived))  // true
        cout << "is Derived" << endl;
    // typeid作用于指针本身时，返回的是该指针的静态类型
    if (typeid(bp) == typeid(dp))  // false
        cout << "No" << endl;
    ```
* type_info类
    * 创建type_info类对象的唯一途径是使用typeid运算符;
    * type_info的操作

        | 操作      | 说明 |
        | ----------- | ----------- |
        | t1 == t2      | 如果type_info对象t1和t2表示同一种类型，返回true,否则返回false       |
        | t2 != t2   | 如果type_info对象t1和t2表示同一种类型，返回false,否则返回true        |
        | t.name()   | 返回type_info对象t的类型名        |
        | t1.before(t2)   |  t1是否在t2之前 bool       |
* 其他注意事项
    * typeid作用于指针所指的对象时, 返回指针所指对象的动态类型
    * typeid作用于指针本身时, 返回的是该指针的静态类型
    * typeid返回的std::type_info删除了拷贝构造，所以只能用引用/指针接收
    * typeid返回值会忽略cv限定符


### 5.2.2、dynamic_cast运算符
* 作用：将基类指针或引用安全的转换为派生类的指针或引用
* 形式：type是类类型，且必须含有虚函数，e必须是一个有效的指针/左值/右值
    ```cpp
    dynamic_cast<type*>(e);
    dynamic_cast<type&>(e);
    dynamic_cast<type&&>(e);
    ```
* 示例：
    ```cpp
    class Base{
    public:
        virtual void print() { std::cout << "Base" << std::endl; }
    };
    class Derived: public Base{
    public:
        void print() { std::cout << "Derived" << std::endl; }
        void A() {cout << "Derived A" << endl;}
    };

    int main(){
        Base* b = new Derived;
        // print()如果不是虚函数 b->print()输出Base
        // print()是虚函数 b->print()输出Derived
        b->print();
        // b->A(); // 错 基类指针虽然指向派生类对象 但是基类指针仍然无法访问派生类的特有接口
        // 将基类指针强制转换为派生类指针 指向派生类对象
        Derived* d = dynamic_cast<Derived*>(b);   
        d->A();  // 正常访问
        Base *bb = dynamic_cast<Base *>(d);    // 转换成功 d是通过b转过来的 再转换bb是没问题的

        Base bbb;
        Derived *ddd = dynamic_cast<Derived *>(&bbb);  // ddd=nullptr 不安全 
        if (d != nullptr) {
            d->print();
            d->A();  // 转换之后可以访问派生类特有对象
        } else {
            std::cout << "dynamic_cast failed" << std::endl;
        } 
        delete b;
        return 0;
    }
    ```
* 其他注意事项
    * 如果转换成功，返回转换后派生类的指针或引用；如果转换失败，则会抛出一个std::bad_cast异常;
    * 对一个空指针void执行dynamic_cast, 则返回所需类型的空指针;
    * dynamic_cast运算符只能用于具有虚函数的继承类之间进行转换，否则编译器会报错;
    * dynamic_cast运算符的运行时开销较大，应该避免过度使用;
    * dynamic_cast运算符下行转换随便转，但是上行转换某些情况下不安全；
        ```cpp
        struct A{存在虚函数};
        struct B: public A{...};
        A a;
        B b;
        A *a_ = dynamic_cast<A*>(&b);   // 上行随便转 安全
        B *b_ = dynamic_cast<B*>(&a);   // 下行不安全 b_指针无法访问B类中的特有成员
        B *bb_ = dynamic_cast<B*>(a_);  // 下行安全 a_本来就是b转换来的 所以bb_指针可以访问B类特有成员
        ```

