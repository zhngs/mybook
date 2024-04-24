# 简单工厂

### 一.图解

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>类图</p></figcaption></figure>

简单工厂接收外部字符串，内部进行字符串判断，返回和字符串对应的对象。

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption><p>时序图</p></figcaption></figure>

### 二.特性

解决的问题：

* 将创建对象的职责转移到一个单独的函数内，降低耦合度。

缺点：

* 每加一个具体的产品，都要加一次if判断，违反了开闭原则，所以只适用于产品类型比较少的情况。
