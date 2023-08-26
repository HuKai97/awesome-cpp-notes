[TOC]
# 条款5、item5、优先考虑auto而非显示类型声明
基础：base7、base8

* 类型的名字太长了
    ```cpp
    template <typename It>
    void f(It b, It e) {
        while (b != e) {
            // typename std::iterator_traits<It>::value_type currVal = *b;
            auto currVal = *b;
            std::cout << currVal << std::endl;
            ++b;
        }
    }
    ```

* c++14之后lambda表达式中的形参也可以使用auto（普通函数不可以） -> 模板函数
    ```cpp
    auto f1 = [](const std::unique_ptr<int> &p1, const std::unique_ptr<int> &p2) {
        return *p1 < *p2;
    };
    auto f2 = [] (const auto &p1, const auto &p2) {
        return *p1 < *p2;
    }; 
    // int f3(auto a) {};  // 错
    ```


* lambda表达式的返回值一定要用auto：因为lambda表达式是匿名函数，而底层会实现一个匿名类，所以根本无法获取得到类的名字，虽然可以使用std::function来写lambda表达式是返回值，但是太长，而且性能有损耗
    ```cpp
    std::function<bool(const std::unique_ptr<int> &, const std::unique_ptr<int> &)> f3 = [] (const auto &p1, const auto &p2) {
        return *p1 < *p2;
    };  
    ```

* unsigned的问题：在不同系统中std::vector< int>::size_type和unsigned的大小是不同的，如果是windows中代码移植到linux，这里很可能会出错，所以一般使用auto更好
    ```cpp
    std::vector<int> v;
    // unsigned sz = v.size();
    auto sz = v.size();
    ```

* 避免因为const类型写错而导致无用的拷贝
    ```cpp
    int a = 10;
    const float &b = a;
    // float tmp = a;  const float &b = tmp;

    std::unordered_map<std::string, int> m{{"hello", 1}, {"world", 2}};
    // const std::pair<const std::string, int>
    // 因为const类型写错实际上在内部还需要一次拷贝操作完成类型转换  耗时
    // std::pair<const std::string, int> m2 = {"hello", 5};
    // const std::pair<std::string, int> &tmp = m2;
    // for(const std::pair<std::string, int> &p : m) {}
    
    for(const auto &p : m){}
    ```