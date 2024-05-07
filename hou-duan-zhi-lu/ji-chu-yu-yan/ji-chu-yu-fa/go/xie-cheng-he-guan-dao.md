# 协程和管道

## 一.goroutine

* go里每一个并发活动叫做goroutine，在两个或多个goroutine的并发程序中，两个函数可以同时执行，可以把goroutine理解成线程，但是操作系统可以开启的goroutine远比线程多
*   goroutine的使用是非常简单的，见如下例子

    ```go
    package main
    ​
    import (
        "io"
        "log"
        "net"
        "time"
    )
    ​
    func main() {
        listener, err := net.Listen("tcp", "localhost:7878")
        if err != nil {
            log.Fatal(err)
        }
    ​
        for {
            conn, err := listener.Accept()
            if err != nil {
                log.Print(err) // e.g., connection aborted
                continue
            }
            go handleConn(conn) // 并发执行
        }
    }
    ​
    func handleConn(c net.Conn) {
        defer c.Close()
        for {
            _, err := io.WriteString(c, time.Now().Format("15:04:05\n"))
            if err != nil {
                return // e.g., client disconnected
            }
            time.Sleep(1 * time.Second)
        }
    }
    ```

## 二.通道

* 通道是可以让一个goroutine发送特定值到另一个goroutine的通信机制

### 1.通道创建

*   每一个通道都是某个具体类型的导管，叫做通道的`元素类型`，可以使用内置函数make来创建通道

    ```go
    ch := make(chan int) //无缓冲通道
    ch := make(chan int, 0) //无缓冲通道
    ch := make(chan int, 3) //容量为3的缓冲通道
    ​
    ch := make(chan<- int) //单向通道，只能发送
    ch := make(<-chan int) //单向通道，只能接收
    ```
* 无缓冲通道上的发送操作会阻塞，直到另一个goroutine执行该通道的接收操作；缓冲通道上的接收操作会阻塞，直到另一个goroutine执行该通道的发送操作。无缓冲通道具有同步功能，所以无缓冲通道也叫同步通道
* 缓冲通道是一个队列，当缓冲通道既不满也不空，发送和接收操作都不会阻塞；队列满，发送阻塞；队列空，接收阻塞
* 对缓冲通道使用内置函数len，会得到通道内元素个数；使用内置函数cap，会得到通道缓冲区的容量

### 2.通道拷贝

* 对通道进行拷贝时，拷贝的是引用，底层都是同一份数据

### 3.通道比较

* 同种类型的通道可以使用==进行比较，当二者是同一数据的引用时，比较值为true

### 4.通道操作

*   通道可以进行`发送，接收，关闭`三种操作

    ```go
    ch <- x //发送语句
    x = <-ch //接收语句并赋值
    <-ch //接收语句，丢弃值
    close(ch) //关闭通道
    ```
* 通道关闭后，任何发送操作将导致程序panic，接收操作会获取所有已经发送的值，直到通道为空，这时任何接收操作会立即完成，同时获取到通道元素类型对应的零值
*   可以通过两种方法来直到通道是否已经关闭，第一种是使用接收操作的变种，会多一个布尔值，当通道关闭并且里面的数据已经读完，该bool值为false；第二种是使用range语法，这个更加常用和方便，接收完最后一个值会自动关闭

    ```go
    x, ok = <-ch //第一种方法
    for x := range ch {
        //第二种方法
    }
    ```
* 通道关闭不是必需的，只有通知接收方goroutine所有消息都发送完了才需要关闭通道，当一个通道不再被访问，垃圾回收器会回收它

### 5.通道广播

* 有这样一种情况，我们需要取消一个事件，但是该事件同时存在于多个goroutine中，我们如何做到取消事件后，能够立刻被其他goroutine检测到？如果能够广播给其他goroutine，那么问题就会解决
*   解决方法就是使用通道的close操作，当一个通道关闭之后，所有接收操作会立刻返回并得到零值。那么我们可以利用通道实现广播能力：不在通道上发送数据，而是关闭通道

    ```go
    var done = make(chan struct{})
    select {
        case <-done:
            //...
    }
    ```
* 注意这里创建了一个类型为`struct{}`的通道，这是空结构体类型，里面没有任何数据，不难想到，这样的通道不是用来发送数据的，而是用来广播的。当关闭done这个通道时，select语句会立刻检测到done接收的分支被触发，就达到了广播的效果

## 三.select

*   考虑这样一种情况，我们需要同时等待多个通道，我们不能只从一个通道上接收，因为会阻塞其他通道。这时候需要一种通道`多路复用`的操作，这就是select

    ```go
    select {
        case <-ch1:
            //...
        case x := <-ch2:
            //...
        case ch3 <- y:
            //...
        default:
            //...
    }
    ```
* select语句有一系列分支和一个可选的默认分支，每个分支指定一次通信。select语句会一直阻塞，只能其中一个分支满足条件，然后select语句退出
* select里面如果没有分支，即select{}会有永久阻塞的效果
* select可以实现管道的非阻塞发送

{% code fullWidth="false" %}
```go
ch := make(chan int, 1) // 创建一个容量为1的管道
ch <- 1                 // 预先发送一个数据

select {
case ch <- 2:
	fmt.Println("数据2发送成功")
default:
	fmt.Println("管道已满，数据2发送失败")
}
```
{% endcode %}
