# push_back的整个过程现在定义一个类

```C++
//定义一个类A
class A{
    int x;
    double y;
};
 
//定义一个空的vector，vector中可以存放的是类A的对象
vector<A> vec;
 
//定义类A的对象a，对象b，对象c和对象d
A a;
A b;
A c;
A d;

```

以下push_back()会发生什么：

* vec.push_back(a);

  一开始vec是空的，所以会调用allocate申请一块能容纳一个A类对象的内存了，并调用拷贝构造函数把a赋值给vec的finish迭代器所指向的内存

* vec.push_back(b);

  现在vec也没有多余的可用空间，所以会调用allocate申请一块能够容纳两个A类对象的内存，把原来vec的唯一元素a调用移动构造函数移动到新的vec上去，并调用拷贝构造函数把b复制给新的vec的finish迭代器指向的内存，这时a和b存放在相邻内存中

* vec.push_back(c);

  现在vec也没有多余的空间，所以会调用allocate申请一块能容纳四个A类对象的内存，把原来的vec的两个元素a和b调用移动拷贝构造到新的vec上去，并调用拷贝构造函数把c赋值给新的vec的finish迭代器所指向的内存，这时内存中还能容纳一个A类对象

* vec.push_back(d);

  现在vec还有一个空间，调用拷贝构造函数把d赋值给finish迭代器指向的内存，这块内存没有剩余空间了
