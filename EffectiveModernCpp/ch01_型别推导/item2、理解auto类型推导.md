[TOC]
# 条款2、理解auto类型推导
基础: item1、item7


## 2.1、万能引用的第二种写法：auto &&
和函数模板的万能引用T&&同理，可以参考item1


## 2.2、列表初始化的auto类型推导
具体原因可以查看item7
```cpp
auto x1 = 27;    // auto=int
auto x2(27);     // auto=int
auto x3{27};     // auto=int  
auto x4 = {27};  // auto=std::initializer_list<int>

auto x5{10, 2.};   // 错 无法缩窄
auto x6{27, 3};    // 错 auto只能解析为int 不能传入多个值
```

## 2.3、{}的函数模板类型推导
```cpp
template<typename T>
void f1(T param) {
    cout << "f1(T param)" << endl;
}

template<typename T>
void f2(std::initializer_list<T> param) {
    cout << "f2(std::initializer_list<T> param)" << endl;
}

f1({1, 2, 3});   // 错 T推导不出initializer_list
f2({1, 2, 3});   // 对 T=int  ParamType=std::initializer_list<int>
```


## 2.4、c++14中auto可以作为函数的返回值，但是规则必须是函数模板的规则,所以无法直接推导为initializer_list
```cpp
auto A() {   // 对 auto = int
    return 0;
}
auto B() {  // 错 
    return {1, 2, 3};
}
```