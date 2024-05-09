# 工厂方法

### 一.go代码

```go
package factorymethod

// Operator 是被封装的实际类接口
type Operator interface {
	SetA(int)
	SetB(int)
	Result() int
}

// OperatorFactory 是工厂接口
type OperatorFactory interface {
	Create() Operator
}

// OperatorBase 是Operator 接口实现的基类，封装公用方法
type OperatorBase struct {
	a, b int
}

// SetA 设置 A
func (o *OperatorBase) SetA(a int) {
	o.a = a
}

// SetB 设置 B
func (o *OperatorBase) SetB(b int) {
	o.b = b
}

// PlusOperatorFactory 是 PlusOperator 的工厂类
type PlusOperatorFactory struct{}

func (PlusOperatorFactory) Create() Operator {
	return &PlusOperator{
		OperatorBase: &OperatorBase{},
	}
}

// PlusOperator Operator 的实际加法实现
type PlusOperator struct {
	*OperatorBase
}

// Result 获取结果
func (o PlusOperator) Result() int {
	return o.a + o.b
}

// MinusOperatorFactory 是 MinusOperator 的工厂类
type MinusOperatorFactory struct{}

func (MinusOperatorFactory) Create() Operator {
	return &MinusOperator{
		OperatorBase: &OperatorBase{},
	}
}

// MinusOperator Operator 的实际减法实现
type MinusOperator struct {
	*OperatorBase
}

// Result 获取结果
func (o MinusOperator) Result() int {
	return o.a - o.b
}
```

使用方式如下：

```go
package factorymethod

import "testing"

// compute是稳定的代码，依赖的是稳定的接口
func compute(factory OperatorFactory, a, b int) int {
	op := factory.Create()
	op.SetA(a)
	op.SetB(b)
	return op.Result()
}

func TestOperator(t *testing.T) {
	var (
		factory OperatorFactory
	)

	factory = PlusOperatorFactory{}
	if compute(factory, 1, 2) != 3 {
		t.Fatal("error with factory method pattern")
	}

	factory = MinusOperatorFactory{}
	if compute(factory, 4, 2) != 2 {
		t.Fatal("error with factory method pattern")
	}
}
```

### 二.c++代码

比较版本一和版本二，版本二虽然代码量更多，但是MainForm变成了一个稳定的结构，出现新的Splitter，只需要新建一个子类和一个工厂即可。满足开闭原则，满足依赖倒置原则。

#### 1.版本一

```cpp
class ISplitter {
public:
    virtual void split()=0;
    virtual ~ISplitter(){}
};

class BinarySplitter : public ISplitter {};
class TxtSplitter: public ISplitter {};
class PictureSplitter: public ISplitter {};
class VideoSplitter: public ISplitter {};

class MainForm {
public:
    void Button1_Click(){
        ISplitter * splitter = new BinarySplitter();//依赖具体类
        splitter->split();
    }
};
```

#### 2.版本2

```cpp
//抽象类
class ISplitter{
public:
    virtual void split()=0;
    virtual ~ISplitter(){}
};

//工厂基类
class SplitterFactory{
public:
    virtual ISplitter* CreateSplitter()=0;
    virtual ~SplitterFactory(){}
};

class BinarySplitter : public ISplitter {};
class TxtSplitter: public ISplitter {};
class PictureSplitter: public ISplitter {};
class VideoSplitter: public ISplitter {};

//具体工厂
class BinarySplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new BinarySplitter();
    }
};

class TxtSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new TxtSplitter();
    }
};

class PictureSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new PictureSplitter();
    }
};

class VideoSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new VideoSplitter();
    }
};

class MainForm {
    SplitterFactory* factory; // 工厂
public:
    MainForm(SplitterFactory* factory) {
        this->factory=factory;
    }
    
    void Button1_Click() {  
	ISplitter * splitter = factory->CreateSplitter(); //多态new
        splitter->split();
    }
};
```

### 三.特性

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

解决的问题：

* 满足开闭原则，因为创建产品的职责由工厂子类负责，这样每来一个新产品，新建一个工厂就行。不需要修改原本代码，只需要写新代码即可。
* 满足依赖倒置原则，高层模块依赖抽象，实现细节依赖抽象。
