# 四种cast转换

* **const_cast**

  const_cast的主要作用就是添加或去除指针或者引用的const属性，但不可以是值类型，该类型转换绝大部分出现在函数重载中，其余场合使用const_cast去除const属性是一个很危险的行为  
  

* **static_cast**

  static即为静态类型转换，在编译时进行类型检查以及类型转换，但没有运行时类型检查来保障安全性，其作用主要有4个

  * 相关类型间的转换，double->int,int->double,float->int 等等

    ```C++
    double pi = 3.14;
    int k = static_cast<int>(pi) 
    ```

  * 子类对象向父类对象的转换，子类对象向父类对象相当于对子类对象进行了一个“切片”，只会保留父类中存在的信息

    ```C++
    Base b = static_cast<Base>(derived_ins);
    ```

    注意：把子类的指针或引用转换成基类表示是安全的，进行的是上行转换是安全的，把父类转换成字类时，进行下行转换，由于没有动态类型检查，所以是不安全的

  * void * 与其他类型指针的转换， operator new 就是使用static_cast将void *转换成指向具体对象的指针

    ```C++
    void *memory = operator new(sizeof(Buz());
    buz = static_cast<Buz *>(memory);
    ```

  * 左值和右值之间的转换，move()方法将一个左值转换成右值，其内部就是使用static_cast实现的  
    

* **dynamic_cast**

  dynamic_cast和static_cast的效果是一样的;在进行下行转换时，dynamic_cast具有类型检查的功能，弥补了static_cast类型不安全的缺陷，比static_cast更安全

  多**用于有虚函数的基类与其派生类之间的转换**，特点是进行运行时检测转换类型是否安全，如果转换失败返回nullptr，**依赖于RTTI技术**，但是有额外的函数开销，所以非必要的时候不使用  
  

* **reinterpret_cast**

  几乎什么都可以转，比如将int转指针，可能会出问题，尽量少用
