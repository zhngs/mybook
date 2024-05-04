# 函数

## 一.函数声明

*   函数声明包含一个名字，一个形参列表，一个可选的返回值列表以及函数体

    ```go
    func name(parameter-list) (result-list) {
        body
    }
    ```
*   函数形参如果连续的形参或返回值都是一样，可以简写

    ```go
    func f(i, j, k int, s, t string)
    func f(i int, j int, k int, s string, t string)
    ```
*   函数的返回值可以命名

    ```go
    func add(x, y int) (sum int)
    ```
*   函数可以有多个返回值

    ```go
    func f(x int, y string) (int, error)
    ```
* 如果发现一个函数有函数声明但是没有函数体，说明这个函数使用其他语言实现的

## 二.匿名函数

*   匿名函数即`闭包`，它是一个表达式，闭包的形式和函数类似，但是没有函数名

    ```go
    func squares() func() int {
        var x int
        return func() int {
            x++
            return x * x
        }
    }
    ​
    func main() {
        f := squares()
        fmt.Println(f()) //1
        fmt.Println(f()) //4
        fmt.Println(f()) //9
        fmt.Println(f()) //16
    }
    ```
*   note：`go语言中变量的生命周期不是由作用域所决定的，而是通过它是否可达来确定的`。下面的两个函数中，f函数返回后，变量x仍然可以使用global访问，所以x必然分配在堆上，这种情况叫做x从f中`逃逸`。当g函数返回时，\*y没有从g中逃逸，所以编译器可以安全地在栈上分配内存。我们可以得到另一个结论：`变量分配在栈还是堆上，不是由var或者new关键字决定的`

    ```go
    var global *int
    func f() {
        var x int
        x = 1
        global = &x
    }
    ​
    func g() {
        y := new(int)
        *y = 1
    }
    ```
* note：说一说变量的作用域，`语法块`是有大括号围起来的一个语句序列，比如一个循环或函数体，语法块内部声明的变量对外部不可见。当编译器遇到一个变量名时，将从最内层的语法块开始寻找其声明，如果在内层语法块找到，即使外部语法块也存在同名变量，也不会被使用。

## 三.变长函数

*   在参数列表最后的类型名称之前使用省略号"..."表示声明一个变长函数，调用这个函数的时候可以传递该类型任意数目的参数

    ```go
    func sum(vals ...int) int {
        total := 0
        for _, val := range vals {
            total += val
        }
        return total
    }
    ​
    sum(1, 2, 3) //6
    ```
*   note："..."也可以用在解构slice上

    ```go
    values := []int{1, 2, 3}
    sum(values...) // 效果和sum(1, 2, 3)相同
    ```

## 四.延迟函数调用

* 在c++中一个非常重要的概念是RAII，典型的例子就是智能指针，智能指针创建的时候会拿到堆上一块内存的指针，当智能指针析构的时候，会自动把内存释放掉，从而省下了程序员手动new和delete的负担
*   go语言中的`延迟函数调用defer`就类似于c++中的RAII，defer语句的执行时机是在函数结束后或者函数执行完return后才会调用，所以叫延迟函数调用。defer的一大用途是管理资源，下面例子展示了如何使用defer管理文件的打开和关闭

    ```go
    func ReadFile(filename string) ([]byte, error) {
        f, err := os.Open(filename)
        if err != nil {
            return nil, err
        }
        defer f.Close()
        return ReadAll(f)
    }
    ```
