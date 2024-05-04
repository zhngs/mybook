# 测试

## 一.测试方法

* 编写后缀名为\_test.go的文件来进行测试，在该文件中，有三种函数需要注意，即`功能测试函数`，`基准测试函数`，`示例函数`
* 功能测试函数是以`Test`前缀命名的函数，用来检测一些程序逻辑的正确性，go test运行测试函数，报告结果是PASS还是FAIL
* 基准测试函数以Benchmark开头，用来测试某些操作的性能，go test汇报操作的平均执行时间
* 示例函数以`Example`开头，用来提供机器检查过的文档

## 二.Test函数

*   测试文件必须导入testing包，测试函数必须以Test开头，且后缀名称必须以大写开头

    ```go
    import "testing"
    func TestName(t *testing.T)
    ```
* 一个测试的好坏需要有指标来进行衡量，其中一个重要的指标就是`覆盖率`。覆盖率是指一个测试套件覆盖待测试包的比例
*   `语句覆盖率`是一种简单且广泛使用的方法之一，一个测试套件的语句覆盖率是指部分语句在一次执行中至少执行一次。go中有一个cover工具可以衡量语句覆盖率

    ```go
    go test -coverprofile=c.out //生成覆盖率文件
    go tool cover -html=c.out -o coverage.html //将覆盖率文件转换成html，便于查看
    ```

## 三.Benchmark函数

*   基准测试是在一定工作负载之下检测程序性能的一种方法，前缀是Benchmark的函数拥有一个\*testing.B参数来提供测试方法，例如提供一个整型N来指定执行次数。这个N刚开始是一个较小的值，会根据运行情况推断出能够稳定运行的足够大的N值

    ```go
    func BenchmarkPalindrome(b *testing.B) {
        for i := 0; i < b.N; i++ {
            IsPalindrome("detartrated")
        }
    }
    ```
*   基准测试输出中有一行打印如下，其中BenchmarkPalindrome是基准测试函数的名字，2是GOMAXPROCS的值，67031889是执行次数，17.87ns是每次执行的耗时，这是67031889次执行取的平均值

    ```
    BenchmarkPalindrome-2           67031889                17.87 ns/op
    ```
*   基准测试函数默认不会执行，需要在go test中加入bench参数指定要运行的基准测试，模式"."匹配所有基准测试函数

    ```
    go test -bench=.
    ```

## 四.Example函数

*   示例函数没有参数没有结果，以Example开头

    ```go
    func ExampleIsPalindrome() {
        fmt.Println(IsPalindrome("detartrated"))
        fmt.Println(IsPalindrome("palindrome"))
        // output:
        // true
        // false
    }
    ```
* 示例函数有三个目的：一是作为文档，举一个好的例子是描述库函数功能最简洁直观的方式；二是可以通过go test进行测试，如果示例函数包含像上述例子中的注释，go test就会将输出内容和注释内容进行比对；三是提供手动实验的代码，例如golang.org上的godoc文档服务器使用go playground让用户在web浏览器上编辑和运行示例函数
