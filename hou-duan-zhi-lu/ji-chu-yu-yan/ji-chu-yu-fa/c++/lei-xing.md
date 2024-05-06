# 类型

## 一.基础类型

### 1.整形

下面是在64位机器上的长度，32位机器上会有变化，不过现在基本都是64位机器，想要精确控制整形长度，可以包含头文件stdint.h，里面有`int32_t`等类型

* `char, unsigned char`，8位
* `short, unsigned short`，16位
* `int, unsigned int`，32位
* `long, unsigned long`，64位
* `long long, unsigned long long`，64位

想要获得数值极限，可以使用数值极限库limits

```
#include <limits>
cout << numeric_limits<int>::min() //-2147483648
cout << numeric_limits<int>::max() // 2147483647
```

### 2.浮点型

* `float`，32位
* `double`，64位
* `long double`，128位

数值极限同样可以使用numeric\_limits

### 3.布尔型

* `bool`，两个值true和false，经过测试是8位长度

## 二.复合类型

### 1.数组

* `T array[N]`，本质是栈上一块连续的内存，数组名本质是个指针

### 2.枚举

* c++98的枚举会造成命名空间污染和隐式转换，并且不能前向声明，应该避免使用
* c++11新增了枚举类，推荐使用

```
enum class Color {
    black,
    white,
    red
};
Color a = Color::black;
```

### 3.共用体

* 共用体基本很少用到，在一些嵌入式的底层硬件寄存器编码中会使用到
* 共用体的长度取决于其内部长度最大的元素，如下面的的共用体长度为64位

```
union threeobj {
    int int_val;
    long long_val;
    double double_val;
}
```

### 4.结构体

* 结构体可以看作权限全部为public的类
* 结构体的长度取决于`内存对齐`，这个很重要，正因为内存对齐的存在，结构体一般不能使用memcmp比较大小

```
struct Person {
    Person(int age);
    int age;
};
```

### 5.类

* 类是c++最重要的数据类型
* 传统的面向对象提倡`封装、继承、多态`，最常用的模式是`public继承+多态`来实现各种抽象，但发展到现代，继承和多态并不算是一种好的编程模型，现代流行的语言如go、rust都不直接提供继承，而是通过别的方法来实现多态
* 我赞同陈硕的观点，继承是一种强耦合，强度仅次于友元，`通过继承和多态来实现的设计模式的低耦合本质上是一种不得已而为之的方法，并不值得提倡`
* 我认为c++最重要的技法是`数据封装 + RAII`，组合远优于继承，通过std::function来提供多态性，无论是初学者还是稍有经验的c++人员都应该避免使用传统的继承加多态

```
class Person {
public:
    Person(int age);
private:
    int age;
}
```

## 三.引用类型

### 1.指针类型

* `T*`，和机器位数相关，64位机器指针的长度是64位
* 裸指针除非在一些特殊的场合，不然应该避免使用，c++11有更好的unique\_ptr和shared\_ptr

### 2.函数指针

* `typedef int (*pf)(int, int)`
* 上面声明了一个返回值为int，参数为两个int的函数指针
* 函数指针是c语言用来实现多态功能的唯一方法，c++虽然也可以使用，但是不提倡，我们有更好的std::function来完美取代函数指针

### 3.字符串

* `const char* str = "hello world"`
* 字符串本质是程序静态存储区内的一块以空字符结尾的连续的内存，体现在程序中就是一个const指针

## 四.标准库中的类型

> 前面的类型都是c++语言内置的类型，想要进行编程远远不够，我们还需要掌握标准库中的好用的类型

### 1.string

* RAII的手法封装字符串，内部的内存布局是堆上的一块连续内存

### 2.vector

* RAII的手法封装数组，内部的内存布局是堆上的一块连续内存

### 3.map和unordered\_map

* map的内存布局是红黑树
* unordered\_map的内存布局是散列表

### 4.set和unordered\_set

* set和map本质上是同一个东西，内存布局是一样的，都是红黑树
* unordered\_set同理，内存布局也是散列表

### 5.其他类型

* c++标准库中还有其他类型，但是不常用，最常用的还是`string、vector、map`
* 其他的类型如dequeue，priority\_queue(底层是堆)，list，tuple等用到可以再查
* 推荐[C++ 参考手册 - cppreference.com](https://zh.cppreference.com/w/cpp)来查询标准库
