[TOC]
# item14、如果函数不抛异常，请使用noexcept
基础：base11

* 定义：c++11中，noexcept说明符指定某个函数不会抛出异常
* 使用：当确定这个函数不会发生异常时，使用noexcept可以帮助编译器进行优化，例如在某些情况下可以避免生成额外的异常处理代码；但是如果一个函数写了noexcept但是还是抛出异常，则程序会调用terminate终止程序，程序崩溃
    ```cpp
    void A() noexcept {}  // 正常用法
    void A() noexcept {new ...;}  // 程序可能直接崩溃
    ```

* 两种特殊用法
    * noexcept(编译期的bool值)：相当于开关noexcept的作用，根据代码来判断这个函数是否会抛出异常，noexcept(true) = noexcept，noexcept(false)=什么也不写
    * noexcept(noexcept(函数func(xxx)))：这个函数抛不抛异常，取决于函数func会不会抛异常，如果func抛出异常，相当于noexcept(false)，反之相当于noexcept(true)

* 一些需要注意的点
    * c++11中内存释放函数（operator delete 和 operator delete[]）和析构函数都是隐式的noexcept（自定义的析构函数也是noexcept的），所以两种函数内部不能抛出异常，否则将自动终止程序
        ```cpp
        class A {
        public:
            A() {}
            ~A() { throw exception(); }
        };

        A a;  // 中止程序 虽然这里没写noexcept 但是编译器默认析构函数仍然是noexcept的
        ```
    * 移动构造和移动赋值如果不声明为noexcept，编译器是不敢用的（因为移动构造和移动赋值如果发生异常，那么程序会崩溃，之前等号右边的数据就会丢失）
        * 实例：std::vector添加新元素如果容量是不够的，那么vector会福根配一个新的更大的内存用于存储新元素，并且之前vector中的元素会被赋值或者移动到新的vector中，但是如果移动操作不是noexcept的，那么vector是不敢用移动操作的
            ```cpp
            struct A
            {
                A(std::string param)
                {
                    std::cout << "A(std::string param)" << std::endl;
                    _data = param;
                }
                A(const A &a)
                {
                    std::cout << "A(const A& a)" << std::endl;
                    _data = a._data;
                }
                A(A &&a) noexcept
                {
                    std::cout << "A(A&& a)" << std::endl;
                    _data = std::move(a._data);
                }

            public:
                std::string _data;
            };

            int main(){
                std::vector<A> vec;
                vec.reserve(2);
                // A(std::string param)
                // A(A&& a)
                vec.push_back(A("hello"));  
                // A(std::string param)
                // A(A&& a)
                vec.push_back(A("world"));
                // A(std::string param)  执行一次A(string)构造函数
                // A(A&& a)   A("nihao")是临时变量 所以执行一次移动构造函数将这个临时变量移动到vec中
                // 移动构造函数没有noexcept
                // A(const A& a)  将前两个元素拷贝到新vector中
                // A(const A& a)
                // 移动构造函数有noexcept
                // A(A&& a)       将前两个元素移动到新vector中
                // A(A&& a)
                vec.push_back(A("nihao"));
                return 0;
            }
            ```
        * C++标准库中使用了大量的swap函数，而这些函数是noexcept的一个绝佳使用场地
            ```cpp
            template<class T, size_t N>
            void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap(*a, *b)));
            ```