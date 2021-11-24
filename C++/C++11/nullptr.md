# nullptr

nullptr是C++11用来表示空指针新引入的常量值，再C++中如果表示空指针语义时建议使用nullptr而不要使用NULL，因为NULL本质上是一个int型的0，而不是一个指针

```C++
void func(void *ptr) {
    cout << "func ptr" << endl;
}
void func(int i) {
    cout << "func i" << endl;
}
void main() {
    func(NULL);//输出func i
    func(nullptr);//输出func ptr
}
```

