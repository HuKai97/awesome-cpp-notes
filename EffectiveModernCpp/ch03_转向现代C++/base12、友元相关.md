[TOC]
# 基础12、友元相关
* 作用：让一个函数/类可以访问另一个类中的私有成员

* 全局函数作友元
    ```cpp
    class Building {
    public:
        std::string m_SittingRoom;
        Building(std::string SittingRoom, std::string BedRoom) : 
            m_SittingRoom(SittingRoom), m_BedRoom(BedRoom) {}
        friend void laoWang(Building &building);
    private:
        std::string m_BedRoom;
    };
    void laoWang(Building &Building) {
        std::cout << Building.m_SittingRoom << std::endl;
        std::cout << Building.m_BedRoom << std::endl;  // 访问私有成员
    }

    Building b("SR", "BR");
    laoWang(b);
    ```
* 类作友元: 类中所有成员函数都可以访问Building类中的私有成员
    ```cpp
    class Building {
    public:
        std::string m_SittingRoom;
        Building(std::string SittingRoom, std::string BedRoom) : 
            m_SittingRoom(SittingRoom), m_BedRoom(BedRoom) {}
        friend class laowang;
    private:
        std::string m_BedRoom;
    };

    class laowang {
    public:
        laowang() {
            std::cout << m_building.m_SittingRoom << std::endl;
            std::cout << m_building.m_BedRoom << std::endl;  // 访问私有成员
        }
    private:
        Building m_building{"sr", "br"};
    };
    ```

* 成员函数作友元：只希望类中的某个成员函数访问Building类，注意类两个类的声明定义顺序
    ```cpp
    class Building;
    class laowang {
    public: 
        laowang();
        void visit();
    private:
        Building *m_building;
    };

    class Building {
    public:
        std::string m_SittingRoom;
        Building(std::string SittingRoom, std::string BedRoom) : 
            m_SittingRoom(SittingRoom), m_BedRoom(BedRoom) {}
        friend void laowang::visit();
    private:
        std::string m_BedRoom;
    };

    laowang::laowang() {
        std::cout << m_building->m_SittingRoom << std::endl;
        std::cout << m_building->m_BedRoom << std::endl;  // 错 无法访问私有成员
    }
    void laowang::visit() {
        std::cout << m_building->m_SittingRoom << std::endl;
        std::cout << m_building->m_BedRoom << std::endl;  // 访问私有成员
    }
    ```