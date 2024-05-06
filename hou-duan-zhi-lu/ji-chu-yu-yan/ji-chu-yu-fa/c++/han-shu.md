# 函数

## 一.普通函数

* `T 函数名(T 参数, T 参数, ...) {}`
* c++函数支持重载，返回值不同不能成为重载决议的条件，参数列表不同可以重载，类中的成员函数中const标志也可以成为重载决议条件(本质还是参数不同，只不过变成了this)

## 二.匿名函数(闭包)

### 1.bind

* bind包含在`functional`头文件中，可以将普通函数和成员函数绑定为一个函数闭包，有一个需要注意的点是bind绑定时会把所有参数拷贝到内部，如果参数中有shared\_ptr，会延长生命周期
* bind目前不推荐使用，因为有更好的lambda表达式，lambda用法更加自然，没有bind诘屈聱牙的占位符

```
int func(int a, int b)
{
    cout << a << b;
}
auto f = bind(func, 1, placeholders::_1);
f(2); // 输出：1，2
```

```
class Test
{
public:
    void print(int a)
    {
        cout << a;
    }
}
Test t;
auto f = bind(&Test::print, &t, 1);
f() // 输出：1
```

### 2.lambda

* lambda推荐使用，用法非常自然
* lambda有两种捕获方式，=按值捕获，&按引用捕获

```
int func(int a, int b)
{
    cout << a << b;
}
auto f = [](int b){
    func(1, b);
};
f(2); // 输出：1，2
```

```
class Test
{
public:
    void print(int a)
    {
        cout << a;
    }
}
Test t;
auto f = [&]{ //按引用捕获t
    t.print(1)
};
f() // 输出：1
```

## 三.function

* function在头文件`functional`中，可以完美代替函数指针，可以接受bind，lambda，函数指针，仿函数作为值
* 用法是`function<返回值(参数列表)>`，匹配返回值和参赛列表相同的函数
* `function和lambda结合可以实现强大的回调功能`

```
int func(int a, int b)
{
    cout << a << b;
}
function<int(int, int)> f = func;
f(1, 2); // 输出：1，2
```
