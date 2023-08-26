[TOC]
# 条款11、优先考虑使用deleted函数而非未定义的私有声明

* c++98是如何禁用拷贝构造和拷贝赋值？
    * c++98会将这两个函数Dion声明写在private中，且不去定义，此时在类外调用这两个函数就会报函数不可访问；不去定义是为了阻止成员函数或类的友元调用，如果强行调用会在链接处报错（不直观 不知道错在哪）
        ```cpp
        class A {
        public:
            A(int data) : _data(data) {}

        private:
            int _data;
            A(const A &a);
            A& operator=(const A&);
            friend class B;
        };

        struct B {
            B() {
                A a(10);
                A aa = a;
            }
        };

        A a(10);
        // A aa = a;  // 不可访问
        B b;   // 链接处报错 未定义
        ```
* c++11使用=delete来禁用相关函数，通常delete掉的函数会写在public中，否则报错信息不明确
    ```cpp
    class A {
    public:
        A(int data) : _data(data) {}
        A(const A &a) = delete;
        A& operator=(const A&) = delete;
    private:
        int _data;
    
        friend class B;
    };

    A a(10);
    A aa = a;  // 不可访问 编译不通过
    ```

* delete可以禁止任何函数，不仅仅删除类中的函数
    ```cpp
    void func(int number) = delete;

    int a = 10;
    func(a);  // 已经删除的函数 不能调用
    ```

* delete还可以禁止一些模板的实例化
    * 实例：假如定义了一个模板函数A,但是我不希望A< void*>的实例函数出现，void类型我想另外处理
        ```cpp
        template <typename T>
        void A(T *ptr) {}

        template<>
        void A(void *ptr) = delete;

        void *p = nullptr;
        A(p);  
        ```
    * 尤其对于类中的函数模板：如果是c++98版本，将模板函数写在public中，特化版本写在private，但是代码会直接报错，因为c++要求两者必须是同一个访问级别的
        ```cpp
        class B {
        public:
            template <typename T>
            void A(T *ptr) {}

            template<>
            void A(void *ptr) = delete;
        };
        ```