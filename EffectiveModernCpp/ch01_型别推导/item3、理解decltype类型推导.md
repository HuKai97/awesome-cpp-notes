[TOC]
# 条款3、理解decltype类型推导
基础: item9、item23

## 3.1、前提知识
* 变量和表达式的区别
    * 变量：变量是用来存储和表示数据的标识符，可以存储不同类型的数据，例如数字、字符串、布尔值等，通过使用变量，我们可以在程序中动态地存储和操作数据。 
    * 表达式：由操作数和运算符组成的组合，用于计算出一个值。表达式可以包含变量、常量、函数调用和运算符等。用于执行各种数学和逻辑运算，以及生成结果。
        * 常量：0,3.14,'A',"hello",true等；
        * 运算符：+，-，*，、，%，++，--，==，&等;
        ```cpp
        int a = 10;   // a是变量也是表达式   
        int *p = &a;  // p是变量也是表达式   *p是表达式   &a是表达式
        // 0是一个表达式
        int sum(int a, int b){ return a + b; }
        sum(3, 5);    // 函数调用表达式 
        ```

## 3.2、decltype(变量)
* 变量的所有修饰都会被保留（包括顶层const、底层const、&、&&、*）;
    ```cpp
    class A{};
    int a = 10;
    decltype(a) b;   // int b;
    const int *const c = &a;
    decltype(c) d = &a;    // const int *const c = &a;
    int &e = a;
    decltype(e) f = a;    // int &f = a;
    int &&g = 0;          
    decltype(g) h = 0;   // int &&h = 0;
    A a1;
    decltype(a1) a2;     // A a2;
    ```

* 数组(vec[i]将返回元素的左值引用)与函数不会发生退化为指针;
    ```cpp
    int array[2] = {1, 2};
    decltype(array) aa;  // int aa[2];  数组不会退化为指针int*
    decltype(array[1]) b = a;  // int &b = a;
    decltype(fun) bb;    // int bb();   函数也不会退化为指针int(*)()
    ```

## 3.3、decltype(表达式)
* 会返回表达式结果对应的类型（所有修饰同样会被保留），如果表达式结果是一个左值，则会得到该类型的左值引用, 相反如果是右值，则会得到该类型;
    ```cpp
    int a = 10;
    int *p = &a;          // 一个指针 指向int变量
    decltype(*p) a1 = a;  // int &a1 = a;
    decltype(p) a2;       // int *a2;
    decltype(a) a3;       // int a3;
    decltype(10) a4;      // int a4;  10是一个左值表达式
    decltype("hello") a5; // const char (&a5)[6];  "hello"是一个右值表达式
    decltype(a + 10) a6;  // int a6;  
    const int b = 10;
    decltype(b+10) b1;
    ```
* 特殊情况1：对于表达式是左值又是变量的情况，decltype默认是用它的变量属性，如果要用它的表达式属性，需要再加一个();
    ```cpp
    int a = 10;
    decltype((a)) a1 = a; // int &a1 = a;
    decltype((a+10)) a2;  // int a2;
    ```
* 特殊情况2：当var是指针类型，且表达式是*var时，则decltype(*var)返回的是指针所指向对象的类型，又因为*var是左值，所以返回指针所指向对象的类型 + &;
    ```cpp
    const int *const b = &a;  // b是一个const指针 指向const int变量
    decltype(b) b1 = &a;  // const int *const b1 = &a;
    decltype((b)) b2 = 0; // const int *const &b2 = 0;
    decltype(*b) b3 = 0;  // const int &b3 = 0;
    int c = 10;
    const int d = 10;
    const int *d1 = &d;
    int *const d2 = &c;
    decltype(*d1) d3 = d;  // const int &d3 = d;
    decltype(*d2) d4 = c;  // int &d4 = c;
    ```

## 3.4、decltype并不会实际计算表达式的值，编译器会分析表达式并得到类型;

* 示例decltype(函数表达式)：返回函数的返回值类型;
    ```cpp
    int& fun() { std::cout << "fun()" << std::endl; return 0;}

    int a = 10;
    decltype(fun()) b = a;  // int &b = a;  编译器并不会指向函数 只会分析函数返回值的类型
    ```

## 3.5、decltype的适用场景：函数返回值是vector[i]的情况
* std::vector<T>对象，使用operator[i]时将会自动返回T&类型，但是对于std::vector<bool>是个例外，此时会返回std::vector<bool>::reference，不一致，所以此时模板函数的返回值必须自动推导;
    ```cpp
    struct Point
    {
        int x, y;
    };
    int v = 1;
    Point p = {0, 0};
    std::vector<int> vec1 = {1, 2, 3};
    decltype(vec1[1]) v1 = v;   // int &v1 = v;
    std::vector<Point> vec2 = {{1, 2}, {3, 4}, {5, 6}};
    decltype(vec2[1]) v2 = p;   // Point &v2 = p;
    std::vector<bool> vec3 = {true, false, true};
    decltype(vec3[1]) v3;       // std::vector<bool>::reference v3;
    ```
* 自动推导
    * c++11：auto + decltype；
        ```cpp
        template<typename T, typename index>
        auto test(T &t, index i) -> decltype(t[i]) {
            return t[i];
        }
        test(vec1, 1) = 0;            // 函数返回值推导为int&  可以正常被修改
        test(vec2, 1) = {1, 1};       // 函数返回值推导为Point&  可以正常被修改
        test_error(vec3, 1) = true;   // 返回值类型推导为std::vector<bool>::reference 可以正常被修改
        ```
    * c++14（错误版本）: 单auto, auto可以作为函数的返回值，但是规则必须是模板的规则；
        ```cpp
        template <typename T, typename index> // C++14版本，
        auto test_error(T &t, index i)        // 不那么正确 因为auto作为模板函数返回值的时候走的是模板推导的套路  
        {
            return t[i]; // 从c[i]中推导返回类型T  没有&
        }
        test_error(vec1, 1) = 0;        // 错 返回值类型是int 函数返回的是右值 错
        test_error(vec2, 1) = {1, 1};   // 返回值类型是Point 右值 但是它执行的是自定义类型的等号运算符重载 并不会修改元素值  即vec2[1] = {3,4}  并没用修改
        test_error(vec3, 1) = true;     // 返回值类型是std::vector<bool>::reference  vec3[1]=true  可以正常修改
        ```
    * C++14（正确但是不够好）：decltype(auto)，等价于c++11版本，可以保留t的所有修饰，但是无法区别&和&&
        ```cpp
        template<typename T, typename index>   // c++14版本
        decltype(auto) test_right(T &t, index i) {   // 可以工作，但是还需要改良
            return t[i];
        }
        // 返回都是引用 全部正常被修改
        test_right(vec1, 1) = 0;        
        test_right(vec2, 1) = {1, 1};   
        test_right(vec3, 1) = true;   
        ```
    * c++14（最佳版本）：decltype(auto) + 万能引用 + std::forward，保留t的所有修饰，并且可用区分&和&&
        ```cpp
        template<typename T, typename index>        // c++14版本
        decltype(auto) test_best(T &t, index i) {   // 最佳版本
            // 传入左值实参 std::forward<T>(t)是左值 [i]后还是左值
            // 传入右值实参 std::forward<T>(t)是右值 但是[i]后还是左值  这个例子不太好 意思明白就行
            return std::forward<T>(t)[i];          
        }
        ```