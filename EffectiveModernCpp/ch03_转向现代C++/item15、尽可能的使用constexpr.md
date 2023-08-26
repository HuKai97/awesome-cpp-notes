[TOC]
# item15、尽可能的使用constexpr
《现代c++语言核心特性解析》p221

constexpr: 可以让很多运算从运行期移动到编译期，好处是可以提高代码运行时的效率，但是需要牺牲编译期的效率，而且当中途去掉constexpr可能造成很多错误


## 15.1、const常量的不确定性
* 编译期常量(可以被编译器读到，被编译器所用，帮助简化程序运行时的计算量)vs运行期常量（编译器无法读到，只能在代码真正运行时才可以执行）
* 某些地方必须传入编译器常量：array模板必须传入一个编译期常量a，因为编译器需要根据a的大小来生成特定的类，所以必须传入一个编译期常量
* 需要的注意的是：本来数组也是需要传入编译期常量的，但是因为gcc编译器可以支持动态设置数组长度，所以数组创建是可以传入一个运行期常量的
    ```cpp
    int e = 10;  // 运行时变量
    // 编译期常量
    const int a = 10;
    const int b = 1 + 2;
    // 运行期常量
    const int c = e;
    const int d = get_value();

    std::array<int, a> arr1;  // 对
    std::array<int, c> arr2;  // 错

    char arr_1[a];
    char arr_2[c];
    ```

## 15.2、constexpr值
* 定义：const修饰的变量并不一定是编译期常量，但是constexpr修饰的变量（必须使用编译期常量/表达式进行初始化）可以保证一定是编译期常量
    ```cpp
    // 运行期变量
    int e = 10;
    // 编译期常量
    const int a = 10;
    const int b = 1 + 2;
    // 运行期常量
    const int c = e;
    const int d = get_value();

    constexpr int aa = 10;
    constexpr int bb = a;
    constexpr int cc = e;  // 错 cc必须使用编译器常量/常量表达式进行初始化
    constexpr int dd = d;  // 错 把运行期常量赋值给编译期常量 
    constexpr int ee = a++;  // 错 a++不是编译器常量
    ```

## 15.3、c++11中constexpr函数
* constexpr修饰一个普通函数
    * (a) 函数的返回值不能是void
    * (b) 函数体只能有一句话，只能是return expr, 其中expr必须是一个常量表达式（编译期常量）
        ```cpp
        constexpr void func1() {}  // 错

        constexpr int func2(int x) {
            // if(x > 0)   // 错
            //     return x;
            // else
            //     return -x;
            return x > 0 ? x : -x;  // 对 只能一句话
        }

        constexpr int func3(constexpr int x) {
            return x > 0 ? x : -x;
        }

        constexpr int func4(int x) {
            return x++;  // 错 x++不是一个编译期常量
        }
        ```
    * (c) 如果给一个constexpr函数一个运行时变量，那么constexpr函数会退化为一个普通函数
        ```cpp
        constexpr int func2(int x) {
            return x > 0 ? x : -x;  
        }
        const int a = 10;  // 编译期常量
        int b = 10;        // 运行时变量
        constexpr int c = func2(a);  // 对  
        constexpr int d = func2(b);  // 错 
        int e = func2(b);  // constexpr函数退化为一个普通函数
        ```

* constexpr修饰构造函数
    * 无constexpr的情况
        ```cpp
        class A {
        public:
            A(): x1(4) {}
            A(int i): x1(i) {}
            int get() const {
                return x1;
            }
            ~A() = default;
        private:
            int x1;
        };
        int a = 10;
        constexpr A a1;  // 错 构造函数A()必须是常量表达式
        constexpr A a1();  // 对 函数声明
        ```
    * constexpr修饰的情况
        * (d) 构造函数初始化列表中传入参数必须是常量表达式constexpr
           
        * (e) 构造函数的函数体必须为空
        * (f) 析构函数必须是默认的，且析构函数必须不能为constexpr
            ```cpp
            class A {
            public:
                constexpr A(): x1(4) {}
                constexpr A(int i): x1(i) {}
                constexpr int get() const {
                    return x1;
                }
                ~A() = default;
                // constexpr ~A() = default;  // 错
            private:
                int x1;
            };

            int a = 10;      // 运行期变量
            constexpr int b = 10;
            constexpr A a1;  // 对
            constexpr A a2(a);  // 必须传入一个编译期常量
            constexpr A a3(b);  // 对
            constexpr A a4(10); // 对
            ```

* constexpr修饰类成员函数
    * (g) constexpr修饰的类成员函数具有const属性
    * (h) 必须使用一个constexpr类对象调用这个constexpr修饰的类成员函数，且用一个constexpr接收这个函数的返回值
        ```cpp
        class A {
        public:
            constexpr A(): x1(4) {}
            constexpr A(int i): x1(i) {}
            constexpr int get() /*const*/ {  // const可以省略
                return x1;
            }
            ~A() = default;
        private:
            int x1;
        };

        constexpr A b(4);
        constexpr int i = b.get();
        ```

## 15.4、c++14中对constexpr
* 打破了a、b、e、g
    ```cpp
    constexpr void func1() {}  // 对 打破a

    constexpr int func2(int x) {  // 对 打破b 
        if(x > 0)   
            return x;
        else
            return -x;
    }

    class A {
    public:
        constexpr A(): x1(4) {   // 对 打破e
            int a = 10;
            a = 50;
        }
        constexpr A(int i): x1(i) {}
        constexpr int get() /*const*/ {  
            x1 = 20;  // 打破g 可以对类的属性作修改 此时如果需要不修改 必须手动添加const
            return x1;
        }
        ~A() = default;
        // constexpr ~A() = default;  // 错
    private:
        int x1;
    };
    ```

* 添加规则：函数可以修改生命周期和常量表达式相同的对象
    ```cpp
    // int x/mm 局部 执行完这个函数死亡  func2也是编译期
    constexpr int func2(int x) {
        int mm = 10;
        mm = 80;
        if(x > 0)   // 对
            return x;
        else
            return -x;
    }
    ```

* 实战：使用constexpr提高代码的运行效率
    ```cpp
    class Point {
    public:
        constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}
        constexpr Point(Point &&p): x(p.x), y(p.y) {}

        constexpr double getX() const noexcept { return x; }
        constexpr double getY() const noexcept { return y; }
        constexpr void setX(double xVal) noexcept { x = xVal; }
        constexpr void setY(double yVal) noexcept { y = yVal; }
    private:
        double x;
        double y;
    };

    // 求一个点关于原点的镜像
    constexpr Point reflection(const Point &p) noexcept {
        Point res;  // res是在编译期创建的 相当于constexpr Point res;
        res.setX(-p.getX());  // 所以res.setX()也在编译器执行
        res.setY(-p.getY());
        return res;
    }

    // 求两个点的中点
    constexpr Point midpoint(const Point &p1, const Point &p2) noexcept {
        return {p1.getX() - p2.getX(), p1.getY() - p2.getY()};
    }

    int main(){
        Point p0(5.6, 8.6);  // 运行时才可拿到
        constexpr Point p1(5.6, 8.6);  // 编译期拿到值
        constexpr Point p2(28.8, 5.4);  // 编译期拿到值
        constexpr auto ref = reflection(p1);   // 编译期拿到函数执行结果
        constexpr auto mid = midpoint(p1, p2); // 编译期拿到函数执行结果
        auto mid2 = midpoint(p0, p2);  // 运行时 因为传入一个运行时参数 
        return 0;
    }
    ```

## 15.5、c++17引入if constexpr

* 使用规则：if constexpr内的代码必须是编译期就可以执行的，当if成立的时候，else将不会被编译
    ```cpp
    void check() {
        if constexpr(sizeof(int) > sizeof(double)) {
            std::cout << "1" << std::endl;
        }else{
            std::cout << "2" << std::endl;
        }
    }
    ```
  