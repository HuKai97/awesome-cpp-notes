[TOC]
# 条款12、使用override声明重写函数

* 重写函数需要满足的条件
    * 基类函数必须是virtual，派生类可以不写virtual。派生类也可以不写override，但是如果不满足这四个条件，编译器是不会报错的
    * 派生类和基类的：函数名(除析构函数)、形参类型、常量性必须完全一致
    * 派生类和基类的：返回值和异常类型必须兼容（只要原来的返回类型是指向基类的指针或引用，新的返回类型是指向派生类的指针或引用，重写的方法就可以改变返回类型，这种类型称为协变返回类型）
    * 引用限定符必须一致(C++11开始)

        ```cpp
        class Base {
        public:
            // 引用限定符必须一致
            virtual void mf1() /*&*/ {
                std::cout << "Base::virtual mf1" << std::endl;
            }
        };

        class Derived: public Base {
        public:
            // 函数名、参数、常量性必须一致
            void mf1() /*const*/ override{
                std::cout << "Derived::virtual mf1" << std::endl;
            }
        };
        ```

* final: 向虚函数添加final可以防止派生类重写，final也可以用于类，表明这个类不能作为基类

* 引用限定符：用于声明一个成员函数必须被左值/右值对象调用
    ```cpp
    class A {
    public:
        using DataType = std::vector<double>;
        // 左值.data()
        DataType& data() &/*引用限定符*/ {
            std::cout << "&" << std::endl;
            return values;
        }
        // 右值.data()
        DataType data() &&/*引用限定符*/ {
            std::cout << "&&" << std::endl;
            return std::move(values);
        }
    private:
        DataType values;
    };

    A makeA() {
        return A();
    }

    A a;
    a.data();  // &
    makeA().data();  // &&
    ```
