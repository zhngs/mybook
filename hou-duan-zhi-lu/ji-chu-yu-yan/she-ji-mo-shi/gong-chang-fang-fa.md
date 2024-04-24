# 工厂方法

### 一.图解

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>类图</p></figcaption></figure>

依赖Product和Factory的代码是稳定的，因为这是两个抽象类。每来一个新产品，就实现一个具体的产品类和工厂类，只需要将该工厂类传入具体的代码即可。

```cpp
int main(int argc, char *argv[]) {
    Factory * fc = new ConcreteFactory();
    Product * prod = fc->factoryMethod();
    prod->use();
    
    delete fc;
    delete prod;
    
    return 0;
}
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption><p>时序图</p></figcaption></figure>

### 二.特性

解决的问题：

* 满足开闭原则，因为创建产品的职责由工厂子类负责，这样每来一个新产品，新建一个工厂就行。不需要修改原本代码，只需要写新代码即可。
