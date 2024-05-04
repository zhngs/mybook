# 方法

## 一.结构体

### 1.结构体字面量

*   结构体字面量有两种书写方式，第一种是按照顺序为每一个成员指定一个值，第二种是指定部分或者全部成员变量的名称和值来初始化结构体变量。第二种初始化方式如果有成员变量没指定，那它的值会是该类型的零值

    ```go
    type Point struct { X, Y int }
    p := Point{1, 2} //第一种
    q := Point{X: 1, Y: 2} //第二种
    ```

### 2.结构体比较

* 如果结构体的所有成员变量都可以比较，那么这个结构体就是可以比较的
* 可以使用==或者!=来进行比较

### 3.匿名成员

*   考虑下面一种情况，将Point嵌套在Circle里，但是有一个问题，访问Point的成员变量变麻烦了，如果有多层嵌套，将会非常痛苦

    ```go
    type Point struct {
        X, Y int
    }
    type Circle struct {
        P Point
        R int
    }
    ​
    var c Circle
    c.P.X = 1
    c.P.x = 2
    ```
*   `匿名成员`可以解决这个问题，go可以定义不带名称的结构体成员，只需要指定类型即可。该类型必须是一个命名类型或者指向命名类型的指针

    ```go
    type Circle struct {
        Point
        R int
    }
    ​
    var c Circle
    c.X = 1
    c.Y = 2
    ```
*   note：匿名成员只是把名字隐藏了，该成员的名字就是类型名，只不过在通过点号访问这些名字的时候可以省略，算是一个语法糖。从包含匿名成员的初始化时也可以看出，和正常的结构体初始化没什么不同

    ```go
    p := Circle{1, 2, 3} //错误！
    p := Circle{X: 1, Y: 2, R: 3}//错误！
    p := Circle{Point{1, 2}, 3}//正确
    ```

## 二.方法声明

*   方法声明和普通函数类似，只是在函数名字前面多了一个方法接收者参数，该参数代表着某个类型实现了该方法，可以通过点号访问。该类型不能是`指针类型`和`接口类型`

    ```go
    type Point struct { X, Y float64 }
    //普通函数
    func Distance(p, q Point) {
        return math.Hypot(q.X-p.X, q.Y-p.y)
    }
    //方法
    func (p Point) Distance(q Point) {
        return math.Hypot(q.X-p.X, q.Y-p.y)
    }
    ```

## 三.方法接收者

* 方法接收者可以写成指针形式，也可以写成非指针形式。方法的接收者同样可以看成一个形参，当形参为指针时，调用该方法仅仅只是传入指针，在方法中修改数据可以影响原数据；当形参不为指针是，调用该方法需要拷贝一份原数据，在方法中修改数据不会影响原数据
*   T类型可以调用接收者为_T的方法，因为编译器会自动取地址；_T可以调用接收者为T的方法，因为编译器会自动将取值

    ```go
    func (p *Point) addptr() {
        p.X += 1;
        p.Y +=1;
    }
    func (p Point) add() {
        p.X += 1;
        p.Y +=1;
    }
    ​
    func main() {
        p := Point{1, 1}
        fmt.Println("p init -- ", p) //p init --  {1 1}
        p.add()
        fmt.Println("p add -- ", p) //p add --  {1 1}
        p.addptr()
        fmt.Println("p addptr -- ", p) //p addptr --  {2 2}
    ​
        q := &Point{3, 3}
        fmt.Println("q init -- ", q) //q init --  &{3 3}
        q.add()
        fmt.Println("q add -- ", q) //q add --  &{3 3}
        q.addptr()
        fmt.Println("q addptr -- ", q) //q addptr --  &{4 4}
    }
    ```
* note：如果结构体内存在匿名成员，调用该匿名成员的方法和访问匿名成员的变量方式相同

## 四.方法变量

*   `方法变量`是将方法绑定到具体的接受者，然后只需要给该方法变量提供实参就可以了。本质上是`方法的闭包`

    ```go
    closure := Point{1, 1}
    f := closure.addptr
    f()
    fmt.Println("closure addptr -- ", closure) //closure addptr --  {2 2}
    ```
