# 观察者

### 一.图解

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption><p>类图</p></figcaption></figure>

观察者模式，上图中具体的观察者不应该依赖于具体的主题类，相关参数在update接口中传入即可。

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>时序图</p></figcaption></figure>

### 二.特性

观察者模式将数据生产者和消费者隔离开，当有多个消费者时，满足开闭原则。
