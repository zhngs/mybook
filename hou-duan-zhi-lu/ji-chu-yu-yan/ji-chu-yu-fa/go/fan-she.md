# 反射

## 一.什么是反射

* 在不知道类型的情况下，可以更新变量、运行时查看值、调用方法以及直接操作布局，这种机制称为反射

## 二.反射的意义

*   考虑如下情况，空接口固然可以接收所有值，但是有一个问题，如果每来一个新的类型，都写一次case，那这个函数永远写不完，使用反射可以解决这个问题

    ```go
    func Sprint(x interface{}) {
        switch x := x.(type) {
            case string:
                //...
            case int:
                //...
            default:
                //...
        }
    }
    ```
* 反射可以增强go语言的表达能力，像encoding/json和encoding/xml提供的编解码功能，text/template和html/template提供的模板机制，go语言的grpc，都是通过反射实现的

## 三.reflect包

> 标准库里面的reflect包实现了反射功能

### 1.reflect.Type

* reflect.Type是一个拥有很多方法的接口，这些方法可以识别类型和透视类型的组成部分，比如结构体的各个字段或函数的各个参数
*   reflect.TypeOf函数接收任何的interface{}参数，并返回reflect.Type类型

    ```go
    t := reflect.TypeOf(3)
    fmt.Println(t) // int
    fmt.Println(t.String()) // int
    ```

### 2.reflect.Value

* reflect.Value是一个结构体，可以包含任意类型的值
*   reflect.ValueOf函数接收任何类型的interface{}函数，并将接口的动态值以reflect.Value的形式返回

    ```go
    v := reflect.ValueOf(3)
    fmt.Println(v) // 3
    fmt.Println(v.String() // <int Value>
    ```
* note：reflect.Value和interface{}都可以包含任意类型的值，二者的区别是interface隐藏了值的信息，除非我们把类型断言出来，否则能做的事情有限；reflect.Value有很多方法去分析包含的值，而且不用知道其类型

## 四.结构体字段标签

*   go允许使用结构体`字段标签`来修改go结构值的json编码方式，这个功能就是依靠反射实现的，反射可以获得结构体的字段标签，以此来进行编解码

    ```go
    package main
    ​
    import (
        "encoding/json"
        "fmt"
        "log"
    )
    ​
    type Movie struct {
        Title  string
        Year   int  `json:"released"`
        Color  bool `json:"color,omitempty"`
        Actors []string
    }
    ​
    var movies = []Movie{
        {Title: "Casablanca", Year: 1942, Color: false,
            Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
        {Title: "Cool Hand Luke", Year: 1967, Color: true,
            Actors: []string{"Paul Newman"}},
        {Title: "Bullitt", Year: 1968, Color: true,
            Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
    }
    ​
    func main() {
        data, err := json.Marshal(movies)
        if err != nil {
            log.Fatalf("JSON marshaling failed: %s", err)
        }
        fmt.Printf("%s\n", data)
    }
    ​
    //output
    [{"Title":"Casablanca","released":1942,"Actors":["Humphrey Bogart","Ingrid Bergman"]},
    {"Title":"Cool Hand Luke","released":1967,"color":true,"Actors":["Paul Newman"]},
    {"Title":"Bullitt","released":1968,"color":true,"Actors":["Steve McQueen","Jacqueline Bisset"]}]
    ```

## 五.警告

* 反射虽然功能和表达能力都很强大，但是要慎用，有三个原因：
* 一是基于反射的代码是很脆弱的，会降低自动重构和分析工具的安全性和准确度，因为无法检测到类型信息
* 二是大量的反射代码难以理解
* 三是基于反射的函数会比为特定类型优化的函数慢一两个数量级
