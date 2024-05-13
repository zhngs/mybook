# 接口

## 一.接口声明

*   接口支持匿名字段，可以将一个接口嵌入到另一个接口中

    ```go
    type Writer interface {
        Write(p []byte) (n int, err error)
    }
    type Reader interface {
        Read(p []byte) (n int, err error)
    }
    type ReadWriter interface {
        Reader
        Writer
    }
    ```

## 二.接口实现

* go的接口是`非侵入式`的接口，只要一个类型实现了某个接口的全部方法，就可以认为该类型实现了该接口
*   某类型实现接口是继承语义，代表该类型是(is-a)特定接口类型，可以直接将该类型赋值给特定接口类型

    ```go
    var w io.Writer
    w = os.Stdout //os.Stdout实现了io.Writer
    ```

## 三.接口赋值

* 接口的内存布局如图所示，要对接口进行赋值，首先某个类型T要实现该接口

<figure><img src="../../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

* 赋值给接口后，接口会将存储该变量的地址，通过该地址才可以调用类型T的方法
* 然后接口会保存赋值的类型，是T形式的还是\*T形式的，这两种形式有不同的方法集
* `T形式的方法集是*T形式的方法集的子集`
*   note：为什么T形式的方法集是\*T形式的方法集的子集？这是精心设计过的，将某结构体按照T形式赋值给接口，语义上是对该变量的一次拷贝，此时该拷贝变量调用接收者为指针形式的方法没有意义，因为如果该方法改变成员变量的值的话，改变的只是拷贝，并不会影响原变量。看完下面的程序会更容易理解这段话：

    ```go
    type Add interface {
        add()
    }
    ​
    type Addptr interface {
        addptr()
    }
    ​
    type Addall interface {
        Add
        Addptr
    }
    ​
    type Point struct {
        X, Y int
    }
    ​
    func (p *Point) addptr() {
        p.X += 1
        p.Y += 1
    }
    func (p Point) add() {
        p.X += 1
        p.Y += 1
    }
    ​
    func main() {
        var a Addall
        p := Point{1, 1}
        a = &p
        fmt.Println("a init -- ", a) //a init --  &{1 1}
        a.add()
        fmt.Println("a add -- ", a) //a add --  &{1 1}
        a.addptr()
        fmt.Println("a addptr -- ", a) //a addptr --  &{2 2}
    ​
        // var b Addall
        // b = p //错误！p并没有实现Addall接口
    ​
        var b Add
        b = &p
        fmt.Println("b init from ptr -- ", b) //b init from ptr --  &{2 2}
        b = p
        fmt.Println("b init -- ", b) //b init --  {2 2}
    ​
        var c Addptr
        c = &p
        fmt.Println("c init from ptr -- ", c) //c init from ptr --  &{2 2}
        // c = p //错误！p没有实现Addptr接口
    }
    ```

## 四.接口比较

* 如果两个接口值都是nil或者二者的动态类型完全一致且动态值相等，那么两个接口值相等，用==和!=号可以比较，也可以作为map的key。需要注意的是，如果接口的动态值本身是不能比较的，那么进行比较会让程序panic
*   note：空接口和动态值为nil的接口是不一样的

    ```go
    var w io.Writer // 此时 w==nil，是空接口
    ​
    var buf *bytes.Buffer // 此时 buf==nil
    w = buf // 此时 w!=nil，不是空接口
    ​
    w = nil // w又回归到nil状态
    ```

## 五.error接口

*   error接口是go语言错误处理的标准模式

    ```go
    type error interface {
        Error() string
    }
    // 下面是完整的errors包
    package errors
    type errorString struct { 
        text string 
    }
    func New(text string) error { 
        return &errorString{text} 
    }
    func (e *errorString) Error() string { 
        return e.text 
    }
    ```
*   直接使用errors.New比较罕见，使用fmt.Errorf调用更为常见

    ```go
    package fmt
    import "errors"
    func Errorf(format string, args ...interface{}) error {
        return errors.New(Sprintf(format, args...)
    }
    ```

## 六.空接口

* 空接口类型为`interface{}`，可以存储任意类型的值。因为接口是空的，所以可以认为任何类型都实现了该接口，所以该接口可以接受任何值

## 七.类型断言

* 使用`x.(T)`的形式可以进行类型断言，其中x是一个接口。T有两种可能：
  * 第一种可能是T是一个具体类型，如果x内部的动态类型为T，则断言成功，返回x的动态值，类型为T，通俗来说就是将值从接口中提取出来；如果断言失败，程序崩溃
  * 第二种可能是T是一个接口类型，如果x内部的动态类型实现了T接口，则断言成功，返回T接口，且内部动态类型和动态值没有发生改变，通俗来说就是进行了接口转换；如果断言失败程序崩溃
*   使用两个结果的赋值表达式的时候断言失败，程序不会崩溃

    ```go
    //断言成功：f为接口内部的提取值，ok为true
    //断言失败：f为断言类型的零值，ok为false
    if f, ok := w.(*os.File); ok {
    }
    ```
