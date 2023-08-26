[TOC]
# 条款6、auto推到若非己愿，使用显示类型初始化惯用法
基础：base9

* 不可见的代理类可能使auto从表达式中推理出"错误的"(不是我们想要的)类型，这时可以使用static_cast强制类型转换强制auto推理出你想要的类型
  

## 6.1、代理类
* 定义：以模仿和增强一些类型的行为为目的类

* 实例：阻止隐式转换的发生
    ```cpp
    class MyArray{
    public:
        class MyArraySize {  // 代理类 模仿和增强int类型
        public:
            MyArraySize(int size) : theSize(size) {}
            int size() const { return theSize; }
            operator int() const { return theSize; }
        private:
            int theSize;
        };

        MyArray(MyArraySize size) : size_(size), data_(new int[size.size()]) {}
        int operator[](int index)
        {
            return data_[index];
        }
        bool operator==(const MyArray &temp)
        {
            return data_ == temp.data_;
        }
        MyArraySize size() { return size_; }

    private:
        int *data_;
        MyArraySize size_;
    };

    class MyArray_ {
    public:
        MyArray_(int size) : size_(size), data_(new int[size]) {}
    private:
        int *data_;
        int size_;
    };

    void fun1(MyArray_ arr) {}
    void fun2(MyArray arr) {}

    int main(){
        // 可以直接调用 因为10可以隐式转换为MyArray_ 但是这样做不好 
        // 所以C++11提出了explict关键字  fun1(MyArray_(10));
        fun1(10);
        // 代理类的作用就体现了 这里会报错 需要转换两次 不允许隐式转换
        // fun2(10);
        MyArray arr(10);
        auto size = arr.size();  // MyArray::MyArraySize
        int size_ = arr.size();  // int 因为代理类中的int()强制转换函数
        return 0;
    }
    ```

## 6.2、临时变量的引用不要用引用来接
* 实例
    ```cpp
    class A
    {
    public:
        A() { std::cout << "A::A()" << std::endl; }
        A(int data) : _data(data) { std::cout << "A::A(int data)" << std::endl; }
        A(const A &a) : _data(a._data)
        {
            std::cout << "A::A(const A &a)" << std::endl;
        }
        A(A &&a) : _data(a._data)
        {
            std::cout << "A::A(A &&a)" << std::endl;
        }
        ~A() { std::cout << "A::~A()" << std::endl; }

        int _data{100};
    };

    std::vector<A> func()
    {
        return {A(1), A(2), A(3), A(4), A(5)};
    }

    std::vector<bool> features()
    {
        return {true, true, true, false, false};
    }

    // func()[2]返回vector[2] 返回一个引用（临时变量） 赋值给a1 引用是直接忽略的
    A a1 = func()[2];
    // func()[3]返回一个引用（临时变量）  再用b去引用这个临时变量 
    // 当执行到下一行时 func()返回的值{A(1), A(2), A(3), A(4), A(5)}已经被全部释放掉了 此时a2将是一个意外的值
    A &a2 = func()[3];
    ```


## 6.3、auto不能用于推断bool类型的临时变量的引用
* 原理：bool类型的底层是通过代理类的引用来实现的
    ```cpp
    std::vector<bool> features()
    {
        return {true, true, true, false, false};
    }
    bool f = features()[4];  // bool
    // bool类型其实底层就是用代理类的引用std::vector<bool>::reference实现的
    // 所以这里直接用auto会用引用类型来接临时变量的引用 报错
    // auto f = features()[4];  // std::vector<bool>::reference  报错
    auto f = static_cast<bool>(features()[4]);
    ```

## 6.4、表达式模板和auto
base9

```cpp
Vec v0 = {1.0, 1.0, 1.0}; // 1000
Vec v1 = {2.0, 2.0, 2.0}; // 1000
Vec v2 = {3.0, 3.0, 3.0}; // 1000
auto v4 = v0 + v1;   // auto才能延迟计算
Vec v3 = v0 + v1 + v2;  // 直接计算 不是延迟计算
auto v4 = static_cast<Vec>(v0+v1+v2);
```