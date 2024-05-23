# context

### 一.接口

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool) // 这个函数给和超时相关的ctx使用
	Done() <-chan struct{} // 这个函数给可取消的ctx使用
	Err() error // Done()返回的chan如果没关闭返回nil，如果关闭返回相关错误
	Value(key any) any // 返回key上的值，注意这个key一般是全局的
}
```

实现上述接口的结构如下：

```go
type emptyCtx int // Background和TODO这两个ctx都是emptyCtx
func (*emptyCtx) Deadline() (deadline time.Time, ok bool) { return }
func (*emptyCtx) Done() <-chan struct{} { return nil }
func (*emptyCtx) Err() error { return nil }
func (*emptyCtx) Value(key any) any { return nil }

// 可取消的ctx
type cancelCtx struct {
	Context
	mu       sync.Mutex            // protects following fields
	done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
	cause    error                 // set to non-nil by the first cancel call
}

// 带超时的可取消的ctx
type timerCtx struct {
	*cancelCtx
	timer *time.Timer // Under cancelCtx.mu.
	deadline time.Time
}

// 携带值的ctx
type valueCtx struct {
	Context
	key, val any
}
```

不同ctx的接口实现如下：

* cancelCtx实现了Value、Done、Err，剩余接口使用内嵌的ctx实现，cancelCtx的Value的实现是用全局唯一的cancelCtxKey来获取cancelCtx自身。
* timerCtx实现了Deadline，剩余接口使用内嵌的cancelCtx实现。
* valueCtx实现了Value，剩余接口使用内嵌的ctx实现。

### 二.Context的树结构

多个Context组合下来一般是树结构，或者是一个单链表（可以看作一种特殊的树），指针是从子节点指向父节点：

* 最顶层一定是emptyCtx。
* 第二层是任意个cancelCtx、timerCtx或valueCtx，以此类推。

### 三.Context树的函数调用逻辑

对于Value函数，会从子节点一直向上找，直到找到满足key的value。

对于Deadline，会从子节点一直向上找timerCtx。

对于Done和Err，会从子节点一直向上找cancelCtx。

### 四.cancelCtx取消子Context

cancelCtx中有一个children字段，用来存储这个节点下面所有的cancelCtx。

父cancelCtx中存储子cancelCtx的原因是，子cancelCtx会向上找第一个cancelCtx当作父节点，将自己注册到父cancelCtx中，这也意味着子cancelCtx和父cancelCtx不一定是相邻的节点，也有可能被valueCtx隔开。

### 五.timerCtx

timerCtx内嵌了cancelCtx，具有一切cancelCtx的特性，和cancelCtx主要有如下不同：

* timerCtx的deadline值一定比父timerCtx小，也就是说，父timerCtx结束子timerCtx一定结束，子timerCtx结束父timerCtx不一定结束。context中会比较timerCtx的父timerCtx的deadline，如果不满足上面的条件，timerCtx会退化成cancelCtx。
* timerCtx停止的时候，会额外停止timer。

### 六.Background

Background是一个全局的ctx，如果很多的请求同时都在使用Background，那么从树的第二层开始，就是一个个不同的请求ctx，相互之间其实是看不到的。
