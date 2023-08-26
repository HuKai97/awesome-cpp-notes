[TOC]
# 基础6、各种类型转换
基础：base5
重要：隐式类型转换、static_cast、dynamic_cast

## 6.1、隐式类型转换
1. 数字相关
    * 整型提升：小整数类型转换为较大的整数类型: bool、char、short等小整型，转换为int、long等大整型
        ```cpp
        bool flag;
        short sval;
        int ival;
        long lval;
        float fval;
        char cval;
        unsigned short usval;
        unsigned int uival;
        unsigned long ulval;
        double dval;

        3.14159L + 'a';  // 'a' -> int -> long double
        dval + ival;     // int -> double
        dval + fval;     // float -> double
        ival = dval;     // double -> int
        flag = dval;     // dval = 0 flag = false else flag = true
        cval + fval;     // char -> int -> float
        sval + cval;     // short -> int, char -> int
        cval + lval;     // char -> int -> long
        // 下面三种不用管，一般不会把无符号数和有符号数相加
        ival + ulval;
        usval + ival;
        uival + lval;
        ```
    * int -> double/float: 小数部分为0
    * double -> int: 默认删除小数部分
  
2. bool类型相关
    * bool类型->非bool类型: false=0, true=1
    * 非bool类型->bool类型: 0=false, 非0=true
    * cin转换为bool类型
        ```cpp
        while(cin >> a)  // 返回cin 再转换为bool类型
        ```
3. 无符号数：无符号数超出范围时，结果是原始值对取值范围n取模后的余数，有符号数超出范围结果是未定义的，可能程序崩溃
4. 字符串：字面串字面值转换成string类型
    ```cpp
    string s = "hello world";
    ```
5. 数组转换为指针：大多数用到数组的表达式中，数组自动会转换为数组首元素的指针
    ```cpp
    int a[10];
    int *p = a;
    ```

## 6.2、强制类型转换
### 6.2.1、static_cast：静态类型转换
1. 内置类型转换；
    ```cpp
    float f = 3.14;
    // int i1 = f;
    int i2 = static_cast<int>(f);
    ```
2. 任意类型指针转换都可以用void*做媒介：可能会导致未定义的行为，要谨慎使用；
    ```cpp
    A a;
    // 对象a的地址强制转换为void类型的指针vp
    void *vp = static_cast<void *>(&a);
    // 再把void类型指针vp强制转换为int类型指针ip
    // 现在可以通过ip操作对象a的内存空间
    int *ip = static_cast<int *>(vp); 
    ```
3. 存在继承的类且没有多态(没有虚函数)的类直接类型转换：上行子类->父类对，下行父类->子类错（原因：子类内容大于父类内容，子类转为父类，直接丢掉多余的部分即可，但是父类转为子类可能发现越界问题）；
    ```cpp
    struct A
    {
        int a{20};
        void funcA() { std::cout << "A::funA" << std::endl; }
    };

    struct CA : public A
    {
        int c{25};
        void funcCA()
        {
            std::cout << "c is: " << c << std::endl;
            std::cout << "CA::funcCA" << std::endl;
        }
    };

    A a;
    CA ca;
    A a_ = static_cast<A>(ca);    // 对
    CA ca_ = static_cast<CA>(a);  // 错
    ```
4. 存在多态的类之间类型转换：上行子类->父类对，但是没有多态（只有父类指针指向子类对象，即发生动态绑定才能能发生多态，这里只发生子类强制转换为父类），下行父类->子类错；
    ```cpp
    struct Base
    {
        virtual void func()
        {
            std::cout << "Base::func()" << std::endl;
        }
    };

    struct Derived : public Base
    {
        virtual void func() override
        {
            std::cout << "Derived::func()" << std::endl;
        }
    };

    Base b;
    Derived d;
    Base b_ = static_cast<Base>(d);
    b_.func();  // Base::func()  没有多态
    // Derived d_ = static_cast<Derived>(b);  // 错
    ```
5. 存在继承但是没有多态的类指针/引用直接转换：上行随便转，不需要void作媒介，但是下行不安全；
    ```cpp
    A a;
    CA ca;
    A *a_ = static_cast<A *>(&ca);    // 没问题
    CA *ca_ = static_cast<CA *>(&a);  // 但是这种不安全
    ca_->funcCA();  // ca_访问c  c is 6422028  这里c不属于ca_ 因为a根本没有
    ```
6. 存在多态的类指针/引用直接转换：随便转，但是某些下行不安全(同dynamic_cast)；
    ```cpp
    Base *b_ = static_cast<Base*>(&d);  // 上行随便转 安全
    Derived *d_ = static_cast<Derived*>(&b);   // 不安全 b_指针无法访问B类中的特有成员
    Derived *dd_ = static_cast<Derived*>(b_);  // 安全 a_本来就是b转换来的 所以bb_指针可以访问B类特有成员
    ```

### 6.2.2、dynamic_cast：动态类型转换
base5

### 6.2.3、const_cast：静态类型转换
* 去掉（指针、引用或指向对象类型成员的指针）对象a的底层const，但是两个变量都是指向同一块区域，还是无法修改对象a的值;
    ```cpp
    const char *pc = "hello world";
    // char *p = pc;  // 错 底层const不能直接删除 不匹配
    char *p = const_cast<char*>(pc);
    p[2] = 'a';  // 错 
    ```
* 唯一用途：函数重载，避免代码重复
    ```cpp
    const std::string &shorterString(const std::string &s1, const std::string &s2)
    {
        return s1.size() <= s2.size() ? s1 : s2;
    }

    std::string &shorterString(std::string &s1, std::string &s2)
    {
        // return s1.size() <= s2.size() ? s1 : s2;
        auto &r = shorterString(const_cast<const std::string &>(s1),
                                const_cast<const std::string &>(s2));
        return const_cast<std::string &>(r);
    }

    string s1 = "hello", s2 = "world";
    string out = shorterString(s1, s2);
    ```

### 6.2.4、reinterpret_cast：静态类型转换
* 作用：通常为运算对象的位模式提供较低层次上的重新解释
* 形式：retinterpret_cast< type>(expression)，其中type和expression至少有一个是指针或引用类型;
* 指针类型之间的转换可以直接用reinterpret_cast，无需void*作媒介，前提是你知道这样转换一定不会出错;
    ```cpp
    int a = 10;
    char *c = "hello wrold";
    int *b = reinterpret_cast<int*>(a);   // int -> int*
    int64_t pci = reinterpret_cast<int64_t>(c);  // char* -> int64_t
    int *d = reinterpret_cast<int *>(&a);  // int* -> int*
    ```