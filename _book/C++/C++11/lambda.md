# lambda

lambda表达式完整的声明格式如下

```C++
[capture list](params list)mutable exception->return type{function body}
```

各项具体含义如下

* capture list:捕获外部变量列表
* params list:形参列表
* mutable指示符:是否可以修改捕获的变量
* exception:异常设定
* return type:返回类型
* function body:函数体

此外，我们还可以省略其中的某些成分来声明不完整的lambda表达式，常见的有

```C++
1.[capture list](params list)->return type {function body}
2.[capture list](params list){function body}
3.[capture list]{fucntion body}
```

* 格式1声明了const类型的表达式，这种类型的表达式不能修改捕获列表中的值
* 格式2省略了返回值类型，但是编译器可以根据以下规则推断出lambda表达式的类型
  如果function body中存在return语句，则该lambda表达式的返回类型由return语句的返回类型确定
  如果function body中没有return语句，则返回值为void类型
* 格式3中省略的参数列表，类时普通函数中的无参函数   
  

**捕获外部变量**

* 值捕获

  值捕获和参数传递中的值传递类似，被捕获的变量的值在lambda表达式创建时通过值拷贝的方式传入，因此随后对该变量的修改不会影响lambda表达式的值:

  ```C++
      int a = 123;
      auto f = [a] {cout << a << endl; };
      a = 321;
      f(); //输出123
  ```

  这里需要注意，如果以传值方式捕获外部变量，则在lambda表达式函数中不能修改该外部变量的值

* **引用捕获**

  使用引用捕获一个外部变量，只需要在捕获列表变量前面加上一个引用说明符&

  ```C++
      int a = 123;
      auto f = [&a] {cout << a << endl; };
      a = 321;
      f(); //输出321
  ```

  引用捕获变量实际上就是该引用所绑定的对象

* **隐式捕获**

  我们可以让编译器根据函数体中的代码来推断需要捕获哪些变量，这种方式称之为隐式捕获，隐式捕获有两种方式，分别是[=]和[&]。[=]表示以值捕获的方式捕获外部变量，[&]表示以引用捕获的方式捕获外部变量

  ```C++
      int a = 123;
      auto f = [&] { cout << a << endl; };    // 引用捕获
      a = 321;
      f(); // 输出：321
  ```



从工程的角度，大部分情况不推荐使用默认捕获符。更一般化的一条工程原则是：显式的代码比隐式的代码更容易维护。一般而言，按值捕获是比较安全的做法，按引用捕获时则需要更小心些，必须确保能被捕获的变量和lambda表达式的生命周期至少一样长，并且在下面需求之一时才能使用：

* 需要在lambda表达式中修改这个变量并让外部观察到
* 需要看到这个变量在外部被修改的结果
* 这个变量的复制代价比较高



### lambda表达式原理

**编译器会把一个lambda表达式生成一个匿名类的匿名对象，并在类中重载函数调用运算符**
