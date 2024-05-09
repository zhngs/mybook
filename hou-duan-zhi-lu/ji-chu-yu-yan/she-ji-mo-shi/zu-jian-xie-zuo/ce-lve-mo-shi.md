# 策略模式

### 一.go代码

```go
package strategy

import "fmt"

type Payment struct {
	context  *PaymentContext
	strategy PaymentStrategy
}

type PaymentContext struct {
	Name, CardID string
	Money        int
}

func NewPayment(name, cardid string, money int, strategy PaymentStrategy) *Payment {
	return &Payment{
		context: &PaymentContext{
			Name:   name,
			CardID: cardid,
			Money:  money,
		},
		strategy: strategy,
	}
}

func (p *Payment) Pay() {
	p.strategy.Pay(p.context)
}

type PaymentStrategy interface {
	Pay(*PaymentContext)
}

type Cash struct{}

func (*Cash) Pay(ctx *PaymentContext) {
	fmt.Printf("Pay $%d to %s by cash", ctx.Money, ctx.Name)
}

type Bank struct{}

func (*Bank) Pay(ctx *PaymentContext) {
	fmt.Printf("Pay $%d to %s by bank account %s", ctx.Money, ctx.Name, ctx.CardID)

}
```

### 二.c++代码

#### 1.版本一

```cpp
enum TaxBase {
	CN_Tax,
	US_Tax,
	DE_Tax,
	FR_Tax
};

class SalesOrder{
    TaxBase tax;
public:
    double CalculateTax(){
        if (tax == CN_Tax){
            //CN***********
        } else if (tax == US_Tax){
            //US***********
        } else if (tax == DE_Tax){
            //DE***********
        } else if (tax == FR_Tax){  //����
	    //...
	}
     }
    
};
```

#### 2.版本二

```cpp
class TaxStrategy{
public:
    virtual double Calculate(const Context& context)=0;
    virtual ~TaxStrategy(){}
};

class CNTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class USTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class DETax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};


class SalesOrder{
private:
    TaxStrategy* strategy;

public:
    SalesOrder(StrategyFactory* strategyFactory){
        this->strategy = strategyFactory->NewStrategy();
    }
    ~SalesOrder(){
        delete this->strategy;
    }

    // 稳定的代码
    public double CalculateTax(){
        Context context();
        
        double val = strategy->Calculate(context);
    }
    
};
```

### 三.特性

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

解决问题：动态使用各种策略，满足开闭原则。

可以发现策略模式和模板方法实际是一样的类图，都是简单的一次接口实现。区别在于模板方法是有一系列规定好的方法按照特定顺序调用，子类实现其中某几个方法，侧重于一系列连续的行为。而策略模式侧重于多种策略存在，重点关注开闭原则。
