# 单例模式

### 一.go代码

```go
package singleton

import "sync"

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

### 二.特性

解决的问题：实现全局只有一个该对象的语义，考虑多线程安全。
