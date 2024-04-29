# 单例模式

### 一.go代码

{% code lineNumbers="true" %}
```go
package singleton

import "sync"

// Singleton 是单例模式接口，导出的
// 通过该接口可以避免 GetInstance 返回一个包私有类型的指针
type Singleton interface {
	foo()
}

// singleton 是单例模式类，包私有的
type singleton struct{}

func (s singleton) foo() {}

var (
	instance *singleton
	once     sync.Once
)

// GetInstance 用于获取单例模式对象
func GetInstance() Singleton {
	once.Do(func() {
		instance = &singleton{}
	})

	return instance
}
```
{% endcode %}

使用方式如下：

{% code lineNumbers="true" %}
```go
package singleton

import (
	"sync"
	"testing"
)

const parCount = 100

func TestSingleton(t *testing.T) {
	ins1 := GetInstance()
	ins2 := GetInstance()
	if ins1 != ins2 {
		t.Fatal("instance is not equal")
	}
}

func TestParallelSingleton(t *testing.T) {
	start := make(chan struct{})
	wg := sync.WaitGroup{}
	wg.Add(parCount)
	instances := [parCount]Singleton{}
	for i := 0; i < parCount; i++ {
		go func(index int) {
			// 协程阻塞，等待channel被关闭才能继续运行
			<-start
			instances[index] = GetInstance()
			wg.Done()
		}(i)
	}
	// 关闭channel，所有协程同时开始运行，实现并行(parallel)
	close(start)
	wg.Wait()
	for i := 1; i < parCount; i++ {
		if instances[i] != instances[i-1] {
			t.Fatal("instance is not equal")
		}
	}
}
```
{% endcode %}

### 二.特性

解决的问题：实现全局只有一个该对象的语义。
