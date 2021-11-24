# RTTI

### 概念

RTTI(Run Time Type Identification)即通过运行时类型识别，程序能够使用基类的指针或引用来检查着这些指针或引用所指的对象的实际派生类型  


### RTTI机制

为什么会出现RTTI这一机制，当类中含有虚函数时，其基类的指针就可以指向任何派生类的对象，这时就有可能不知道基类指针到底指向的是哪个对象的情况，类型的确定要在运行时利用运行时类型标识做出。为了获得一个对象的类型可以使用typeid函数，该函数反回一个对type_info类对象的引用，要使用typeid必须使用头文件typeinfo


### typeid和dynamic_cast操作符

RTTI提供了两个非常有用的操作符：`typeid`和`dynamic_cast`

* typeid

  它指出指针或引用指向的对象的实际派生类型。主要作用就是让用户知道当前的变量是什么类型的

  * 对于c++的内置数据类型，typeid可以方便的输出它们的数据类型

    ```C++
    #include <iostream>
    #include <typeinfo>
    using namespace std;
    
    int main()
    {
         short s = 2;
         unsigned ui = 10;
         int i = 10;
         char ch = 'a';
         wchar_t wch = L'b';
         float f = 1.0f;
         double d = 2;
    
         cout<<typeid(s).name()<<endl; // short
         cout<<typeid(ui).name()<<endl; // unsigned int
         cout<<typeid(i).name()<<endl; // int
         cout<<typeid(ch).name()<<endl; // char
         cout<<typeid(wch).name()<<endl; // wchar_t
         cout<<typeid(f).name()<<endl; // float
         cout<<typeid(d).name()<<endl; // double
    
         return 0;
    }
    ```

  * 对于自己创建的类对象，依然可以输出它们的数据类型

    ```c++
    #include <iostream>
    #include <typeinfo>
    using namespace std;
    
    class A
    {
    public:
         void Print() { cout<<"This is class A."<<endl; }
    };
    
    class B : public A
    {
    public:
         void Print() { cout<<"This is class B."<<endl; }
    };
    
    struct C
    {
         void Print() { cout<<"This is struct C."<<endl; }
    };
    
    int main()
    {
         A *pA1 = new A();
         A a2;
    
         cout<<typeid(pA1).name()<<endl; // class A *
         cout<<typeid(a2).name()<<endl; // class A
    
         B *pB1 = new B();
         cout<<typeid(pB1).name()<<endl; // class B *
    
         C *pC1 = new C();
         C c2;
    
         cout<<typeid(pC1).name()<<endl; // struct C *
         cout<<typeid(c2).name()<<endl; // struct C
    
         return 0;
    }
    ```

  * RTTI 核心

    ```c++
    #include <iostream>
    #include <typeinfo>
    using namespace std;
    
    class A
    {
    public:
         void Print() { cout<<"This is class A."<<endl; }
    };
    
    class B : public A
    {
    public:
         void Print() { cout<<"This is class B."<<endl; }
    };
    
    int main()
    {
         A *pA = new B();
         cout<<typeid(pA).name()<<endl; // class A *
         cout<<typeid(*pA).name()<<endl; // class A
         return 0;
    }
    ```

  分析：

  * 我使用了两次typeid，但是两次的参数是不一样的；输出结果也是不一样的；当我指定为pA时，由于pA是一个A类型的指针，所以输出就为class A * ；
  * 当我指定*pA时，它表示的是pA所指向的对象的类型，所以输出的是class A；*
  * *所以需要区分typeid(*pA)和typeid(pA)的区别，它们两个不是同一个东西；

  **但是明明pA指向的是B，得到的却是A**

  **划重点：**
   好了，我将Print函数变成了虚函数，输出结果就不一样了，这说明什么？

  * 这就是RTTI在捣鬼了，**当类中不存在虚函数时**，typeid是编译时期的事情，也就是静态类型，就如上面的cout<<typeid(*pA).name()<<endl;输出class A一样；*
  * **当类中存在虚函数时**，typeid是运行时期的事情，也就是动态类型，就如上面的cout<<typeid(*pA).name()<<endl;输出class B一样，关于这一点，我们在实际编程中，经常会出错，一定要谨记。

  **(这个真的很重要 一定要多看看 一个类里面有virutal 和没有virtual 对于编译器来说，做的事完全不同的事情，所有一定要看清楚这个类有没有virtual)**

  

* dynamic_cast

  它允许在运行时刻进行类型转换，从而使程序能够在一个类层次结构安全地转换类型。dynamic_cast提供了两种转换方式，把基类指针转换成派生类指针，或者把指向基类的左值转换成派生类的引用

  当类中存在虚函数时，编译器就会在类的成员变量中添加一个指向虚函数表的vptr指针，每一个class所关联的type_info object也经由virtual table被指出来，通常这个type_info object放在表格的第一个slot。当我们进行dynamic_cast时，编译器会帮我们进行语法检查。如果指针的静态类型和目标类型相同，那么就什么事情都不做；否则，首先对指针进行调整，使得它指向vftable，并将其和调整之后的指针、调整的偏移量、静态类型以及目标类型传递给内部函数。其中最后一个参数指明转换的是指针还是引用。两者唯一的区别是，如果转换失败，前者返回NULL，后者抛出bad_cast异常

  

  

