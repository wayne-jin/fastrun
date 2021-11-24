# default&delete

c++11引入了default特性，多数时候用于声明构造函数为默认构造函数，如果类中有了自定义的构造函数，编译器就不会生成默认构造函数

```C++
class A {
public:
    A(int i) { a = i; }
    int a;
};
void main() {
    A a; //编译出错
}
```

上述代码编译出错，因为没有匹配的构造函数，编译器没有生成默认的构造函数，而程序员只需在函数声明后加上 = default 就可将函数声明为defaulted函数，编译器将为显式声明的defaulted函数自动生成函数体

```C++
class A {
public:
    A() = default;
    A(int i) { a = i; }
    int a;
};
void main() {
    A a; 
}
```

delete:如果开发人员没有定义特殊成员函数，那么编译器在需要特殊成员的时候会隐式自动生成一个默认的特殊成员函数，而我有时候想禁止对对象的拷贝或者赋值，可以用delete进行修饰

```C++
class A {
public:
    A() = default;
    A(const A&) = delete;
    A& operator = (const A&) = delete;
    int a;
    A(int i) { a = i; }
};
void main() {
    A a1;
    A a2 = a1;//错误，拷贝构造函数被禁用
    A a3;
    a3 = a1;//错误，拷贝赋值操作符被禁用
}
```

