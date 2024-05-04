# 并发

## 一.竞态(race condition)

* 对于并发编程，race condition是必然会出现的，我们需要了解其出现的原因：`当多个goroutine同时读写同一块内存，就会产生race condition`
* 解决race condition有较多手段，最常用的方法就是加锁

## 二.内存同步

*   考虑如下代码片段，从直觉上来讲会有四种情况，但其实会出现两种令人困惑的情况

    ```go
    var x, y int
    // goroutine1
    go func() {
        x = 1
        fmt.Print("y:", y, " ")
    }
    // goroutine2
    go func() {
        y = 1
        fmt.Print("x:", x, " ")
    }
    ​
    //直觉上认为正确的四种情况
    y:0 x:1
    y:1 x:1
    x:0 y:1
    x:1 y:1
    //但其实有可能出现如下情况
    x:0 y:0
    y:0 x:0
    ```
* 上述情况的原因是：现代的计算机一般有多个处理器，每个处理器都有本地缓存，为了提高效率，对内存的写入是先缓存到处理器中的，只有在必要的时候才刷回内存，甚至刷回内存的顺序有可能和写入顺序不一样。而在上述情况中，假如goroutine1和goroutine2分别在两个cpu上执行，那么当goroutine1执行x=1时，goroutine2不一定能够观测到x=1，所以就会产生都等于0的情况
* 这种问题都可以采用成熟而简单的手段来避免，比如把变量限制到goroutine中，或者对共享的变量加锁

## 三.互斥锁

*   互斥锁的使用非常简单，只需要掌握Lock和Unlock方法即可。下面的Balance函数演示了如何使用互斥锁保护一个变量

    ```go
    var (
        mu sync.Mutex //用来保护balance
        balance int
    )
    ​
    func Balance() int {
        mu.Lock()
        defer mu.Unlock
        return balance
    }
    ```

## 四.读写互斥锁

* 考虑上述的例子，balance的值并没有改变，只是将其读出来，如果同时有大量的goroutine去调用Balance函数，会变成完全串行的操作，但实际上balance的值如果不改变的话，完全可以并行去读
*   go的sync.RWMutex实现了读操作可以并行执行，但写操作一段时间只能有一个锁的获取者。想要获取读锁，调用RLock()和RUnlock()，想要获取写锁，调用Lock()和Unlock()

    ```go
    var (
        mu sync.RWMutex //用来保护balance
        balance int
    )
    ​
    func Balance() int {
        mu.RLock()
        defer mu.RUnlock()
        return balance
    }
    ```
* 只有当绝大部分goroutine都在竞争读锁时，RWMutex才有优势，锁竞争不激烈的时候比普通的互斥锁是慢的
