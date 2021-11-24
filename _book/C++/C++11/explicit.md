# explicit

explicit专用于修饰构造函数，表示只能显式构造，不可以被隐式转换

不用explicit:

```c++
class A {
public:
    A(int n) {
        cout << "int" << endl;
   }
    A(const char* p) {
        cout << "char" << endl;
    }
};
void main() {
    A a = 'f';;
}
```

执行结果会打印int,这里存在隐式类型转换，让char类型转成了int

修改后：

```C++
class A {
public:
    explicit A(int n) {
        cout << "int" << endl;
   }
    explicit A(const char* p) {
        cout << "char" << endl;
    }
};
void main() {
    A a = 'f';// error，不可以隐式转换
}
```

