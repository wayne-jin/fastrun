# overrride&final

final用于修饰一个类，表示禁止该类进一步派生和虚函数进一步重载

```C++
class Base final{
    virtual void func() {
        cout << "base" << endl;
    }
};

class Derived : Base {
    void func() override {
        cout << "derived" << endl;
    }
};
```

override用于修饰派生类中的成员函数，标明该函数重写了基类函数，如果一个函数声明了override但父类却没有这个虚函数，编译报错，使用override关键字可以避免开发者再重写基类函数时无意产生的错误

```C++
class Derived : Base {
    void func() override {
        cout << "derived" << endl;
    }

    void func1() override {} //基类没有定义func1()，不可以被重写
};
```

