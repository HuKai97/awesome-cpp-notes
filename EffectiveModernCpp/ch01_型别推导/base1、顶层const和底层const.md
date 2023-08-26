[TOC]
# 基础1、顶层const与底层const
## 1.1、定义
* 顶层const：指针本身是一个常量，const修饰的是指针本身，即int * const p，也就是指针常量
* 底层const：指针所指的对象是一个常量，const修饰的是指针所指的对象，即const int *p，也就是常量指针

## 1.2、顶层和底层的判断规则
* 对指针而言：修饰指针本身的是顶层，修饰指针所指的对象的是底层
* 对一般的变量而言都是顶层const
```cpp
const int ci = 42;   // 顶层const
```
* 对引用而言
    * 引用的const（常量引用）都是底层const
        ```cpp
        const int &r = ci; 
        const int &c = 42;
        ```
    * 对一个顶层const取&就变成了底层const
        ```cpp
        const int ci = 42;     // 顶层const
        const int *p2 = &ci;   // 底层const
        ```
## 1.3、一条准则
* 当指向对象的拷贝操作时，顶层const不受什么影响（可以忽略不看），底层const必须一致才能成功赋值
    ```cpp
    int i = 0
    const int ci = 42;   // 顶层const
    // ci是顶层const 但是对一个顶层const取地址&相当于又加了一个底层const
    const int *p2 = &ci; // 底层const  
    const int *const p3 = p2;  // 前面底层const  后面顶层const
    i = ci;  // 正确：ci是顶层const 直接忽略 两者都是int类型
    int *p = p3; //错误  p3=const int*  p=int*  
    p2 = &i  // 正确 常量指针重新指向一个非常量对象
    ```
## 1.4、其他注意点
1. 引用不是对象，没有赋值这个概念，const引用不适合这条准则;
    ```cpp
    const int a = 42;
    const int &b = a;  // 这里是引用 所以const即使是底层const也没关系 不遵循这条准则
    ```

2. 常量引用的右边可以是任意的，可以是非常量、字面量、一个表达式、不同类型的引用等（常规的引用左右两边类型必须完全一致）,但是常量引用非常量时，不能通过修改常量的值来改变非常量的值;
    ```cpp
    // 常量引用非常量
    // 此时编译器会自动匹配 const int temp = 1; const int &b = temp;
    int a = 1;
    const int &b = a;
    // 常量引用右边可以是任何东西
    const int &c = 42;  // 初始化为42
    const int &d = b * 2;  // 初始化为表达式
    // 不同类型的引用
    // 编译器
    // const int temp = e;
    // const int &f = temp;
    double e = 3.14;  // 3.14
    const int &f = e; // 3
    int &f = e;  // 错 普通的引用等号两边必须类型一致

    // 常量引用非常量时，不能通过修改常量的值来改变非常量的值
    // a可变 b可变 可以通过改变b的值来修改a的值
    // c不可变 无法通过c来修改a和b的值
    int a = 1;
    int &b = a;  // b绑定了a
    const int &c = a;  // c也绑定了a  但是不允许通过c改变a 
    a = 2;   // 可以改变a
    std::cout << a << b << c << std::endl; // 2 2 2
    b = 3;   // 可以改变b
    std::cout << a << b << c << std::endl; // 3 3 3
    c = 4;   // c无法改变
    ```

3. 非常量可以等于常量或者常量引用(本质是赋值操作)，但是非常量无法引用常量或常量引用;
    ```cpp
    const int a = 10;
    const int &b = a;
    int c = a;   // 对 非常量=常量 赋值操作 后续对c的修改不会改变a
    int d = b;   // 对 非常量=常量引用 赋值操作 同上
    int &e = a;  // 错 非常量无法引用常量  e是a的别名 a不可变 e可变 矛盾
    int &e = b;  // 错 非常量无法引用常量引用 同上
    ```