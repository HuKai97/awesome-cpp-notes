[TOC]
# 条款23、理解std::move和std::forward
基础: 条款9

## 23.1、std::move是如何实现的
* std::move的作用：不管输入一个左值(将一个左值转换为一个将亡值，将亡值属于右值)还是右值，统统转换为一个右值;


* c++11模拟实现和工作原理
    * 模拟实现
        ```cpp
        template<typename T>
        typename std::remove_reference<T>::type&& move1(T&& param){
            return static_cast<typename std::remove_reference<T>::type&&>(param);
        }

        string&& a = move1(string("hello"));   // 1
        string s = "hello";
        string&& b = move1(s);    // 2
        ```
    * 工作原理
        * 1处：传入一个右值，推断出T=string，函数参数t类型是string&&，接受一个右值，函数返回类型是string&&，return语句中，param是string&&，返回类型是string&&,所以static_cast不作任何处理;最终实例化函数string&& move(string &&t);
        * 2处：传入一个左值，推断出T=string &，函数参数t类型是string& &&，引用折叠为string&，接受一个左值，函数返回类型是string&&，return语句中，param是string&，返回类型是string&&，static_cast发生显示类型转换；最终实例化函数string&& move(string &t);


* c++14实现方式
    ```cpp
    template<typename T>
    decltype(auto) move2(T&& param){
        return static_cast<std::remove_reference_t<T> &&>(param);
    } 

    string&& c = move2(string("hello"));   
    string j = "hello";
    string&& d = move2(i);   
    ```
* 几个注意的点
    * static_cast可以显示的将一个左值转换为右值引用;
    * std::remove_reference<T>，类型萃取器，移除T中的所有引用符号(&和&&);
    * ::type提取当前类型名，因为模板中用到域作用符，所以要用typename确定type是类型成员而非数据成员等其他非类型成员;
    * std::remove_reference_t<T>等价于typename std::remove_reference<T>::type;

* 移动语义不移动：移动语义本质只是将左值对象转换为右值对象，真正的移动操作会在后续的移动构造函数和或拷贝构造函数（std::move后不一定移动，可能执行构造）中进行; std::move只会将左值变量改为右值变量，但是不会影响变量本身的属性如const属性;
    ```cpp
    struct A{
        A(int v): value(v) {std::cout << "A(int v)" << std::endl;}  // 普通构造
        A(const A &a){   // 拷贝构造
            std::cout << "A(const A &a)" << std::endl;
            value = a.value;
        }
        A(A &&a){        // 移动构造
            std::cout << "A(const A &&a)" << std::endl;
            value = a.value;
            a.value = 0;
        }
        int value;
    };

    class B{
    public:
        explicit B(const A a): a_param(std::move(a)) {}  // explicit: 不允许隐式转换
    private:
        A a_param;
    };

    // 首先调用普通构造函数 int 10->构造A a，执行  再std::move(a)->右值 且a仍然是const类型
    // 匹配移动构造函数时 A &&a = const A a;  报错 匹配失败必须是const A &&a = const A a才可以
    // 匹配拷贝构造函数 const A &a = const A a; 成功 执行
    B b{10};  
    ```


## 23.3、初探std::forward
* 联系：std::move和std::forward都可以将左值转换为右值；
* 区别：std::move总是将任何值都转换为右值，std::forward是有条件的std::move，只有当传入一个右值实参时，才会转换为右值;
    ```cpp
    struct A{
        A(int v): value(v) {std::cout << "A(int v)" << std::endl;}  // 普通构造
        A(const A &a){   // 拷贝构造
            std::cout << "A(const A &a)" << std::endl;
            value = a.value;
        }
        A(A &&a){        // 移动构造
            std::cout << "A(const A &&a)" << std::endl;
            value = a.value;
            a.value = 0;
        }
        int value;
    };

    void process(A &lval){
        std::cout << "process(A &lval)" << std::endl;
    }

    void process(A &&rval){
        std::cout << "process(A &&rval)" << std::endl;
    }

    template<typename T>
    void test(T &&param){
        // 虽然万能引用可用自动判断T和param的类型 但是却失去了传入实参的类型 
        // 因为不管传入的是左值还是右值 param都是左值 为了解决这个问题提出了std::forward
        // process(param);   // 只能传入左值
        // process(std::move(param));  // 只能传入右值
        process(std::forward<T>(param));  // 传入实参是左值process就传入左值 右值同理
    }

    A a{10};
    // 传入一个左值实参 T=int& param=int& std::forward不作处理 process直接传入左值param
    test(a);             // process(A &lval)
    // 传入一个右值实参 T=int param=int&&  std::forward将左值param转换为右值 再传入process
    test(std::move(a));  // process(A &&rval)
    ```