# 抽象工厂

### 一.图解

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption><p>类图</p></figcaption></figure>

抽象工厂是对工厂方法的拓展，主要用于多种抽象产品创建。比如产品A是手机，产品B是电脑，工厂1是小米，工厂2是华为，这样每有一类新的产品，比如苹果的手机和电脑，只需要新建一个苹果的工厂。

```cpp
int main(int argc, char *argv[]) {
	AbstractFactory * fc = new ConcreteFactory1();
	AbstractProductA * pa =  fc->createProductA();
	AbstractProductB * pb = fc->createProductB();
	pa->use();
	pb->eat();
	
	AbstractFactory * fc2 = new ConcreteFactory2();
	AbstractProductA * pa2 =  fc2->createProductA();
	AbstractProductB * pb2 = fc2->createProductB();
	pa2->use();
	pb2->eat();
}
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>时序图</p></figcaption></figure>

### 二.特性

解决问题：对工厂方法进行拓展，支持多类产品。

缺点：

* 不支持增加新类型的产品，否则需要更改工厂接口。
