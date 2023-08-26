[TOC]
# 条款13、优先考虑const_iterator而非iterator

* const_iterator：==const迭代器，当一个迭代器值不会发生改变时，可以定义为const iterator==
    * vector扩容（顺序）：当vector超出容量n，再插入一个新的元素，c++会在另一个内存地址开辟一个2n大小的空间，将原空间中的n个数拷贝/移动过来，再插入新的元素
        ```cpp
        // =======使用iterator
        std::vector<int> vec1 = {1, 2, 3, 4, 5};
        std::vector<int>::iterator it = std::find(vec1.begin(), vec1.end(), 2);
        vec1.insert(it, 22);
        // =======使用const_iterator
        std::vector<int> vec2 = {1, 2, 3, 4, 5};
        std::vector<int>::const_iterator cit = std::find(vec2.cbegin(), vec2.cend(), 2);
        vec2.insert(cit, 22);  // {1,22,2,3,4,5}
        ```
* c++98的const_iterator非常不好用
    * c++98标准库不提供vec.cbegin()和vec.cend()成员函数，虽然可以使用static_cast将vec.begin()和vec.end()生成的iterator转成const_iterator，但是如果想向vector中插入新元素，此时vec.insert函数只能支持iterator，而const_iterator不能转换为iterator，所以c++98的const_iterator非常难用，但是c++11之后这些问题都解决了
* c++11之后，c++14之前，cbegin和cend的没有函数形式的实现，但是可以通过向begin和end函数中传入const容器来实现；
    ```cpp
    std::vector<int> vec1 = {1, 2, 3};
    auto it1 = vec1.cbegin();  // std::vector<int>::const_iterator
    // auto it2 = std::cbegin(vec1);   // 错 c++11没有cbegin()函数  c++14才开始有
    std::cout << *it1 << std::endl;  // 1

    const std::vector<int> vec2 = {1, 2, 3};
    auto cit1 = std::begin(vec2);  // std::vector<int>::const_iterator
    std::cout << *cit1 << std::endl;  // 1
    ```