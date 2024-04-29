# 简单工厂

### 一.go代码

简单工厂外部可以传入一个值，然后内部判断，以此来构造不同的对象。

{% code fullWidth="false" %}
```go
package simplefactory

import "fmt"

// API is interface
type API interface {
	Say(name string) string
}

// NewAPI return Api instance by type
func NewAPI(t int) API {
	if t == 1 {
		return &hiAPI{}
	} else if t == 2 {
		return &helloAPI{}
	}
	return nil
}

// hiAPI is one of API implement
type hiAPI struct{}

// Say hi to name
func (*hiAPI) Say(name string) string {
	return fmt.Sprintf("Hi, %s", name)
}

// helloAPI is another API implement
type helloAPI struct{}

// Say hello to name
func (*helloAPI) Say(name string) string {
	return fmt.Sprintf("Hello, %s", name)
}
```
{% endcode %}

以下是使用方式：

```go
package simplefactory

import "testing"

// TestType1 test get hiapi with factory
func TestType1(t *testing.T) {
	api := NewAPI(1)
	s := api.Say("Tom")
	if s != "Hi, Tom" {
		t.Fatal("Type1 test fail")
	}
}

func TestType2(t *testing.T) {
	api := NewAPI(2)
	s := api.Say("Tom")
	if s != "Hello, Tom" {
		t.Fatal("Type2 test fail")
	}
}
```

### 二.特性

解决的问题：

* 将创建对象的职责转移到一个单独的函数内，降低耦合度。

缺点：

* 每加一个具体的产品，都要加一次if判断，违反了开闭原则，所以只适用于产品类型比较少的情况。
