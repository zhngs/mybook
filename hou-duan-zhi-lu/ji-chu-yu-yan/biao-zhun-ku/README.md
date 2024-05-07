# 标准库

go官方标准库：[https://pkg.go.dev/std](https://pkg.go.dev/std)

### 一.分类

#### 1.数据结构相关

* bytes，字节切片相关函数，和strings包类似。
* strings，操作utf8字符串的库。
* slices，切片的相关函数。
* strconv，和string相互转换的函数。
* cmp，和比较相关的包。
* sort，排序相关库。
* container，额外的容器，如heap、list、ring。
* unicode，unicode相关库。

#### 2.IO相关

* io，提供io相关的基本接口。
* bufio，实现缓冲 I/O。 包装一个 io.Reader 或 io.Writer 对象，创建另一个对象（Reader 或 Writer），该对象也实现该接口，为文本 I/O 提供缓冲和一些其他帮助。
* fmt，类似于 C 语言的 printf 和 scanf 的函数来实现格式化 I/O。

#### 3.网络相关

* net，网络库。
* compress，提供若干压缩算法。
* encoding，编解码相关库。
* crypto，密码库，提供多种算法。
* hash，哈希算法库。
* html，提供了转义和取消转义 HTML 文本的函数。
* database，sql相关接口。

#### 3.底层相关

* builtin，内置包。提供一些内置类型，函数。
* os，操作系统的功能接口。
* flag，命令行参数解析。
* path，路径库。
* reflect，反射库。
* unsafe，绕过go安全类型安全的操作。

#### 4.协程相关

* context，协程之间控制退出或传值。
* sync，协程同步库。
* time，时间相关库。

#### 5.其他

* embed，提供对正在运行的 Go 程序中嵌入的文件的访问。
* errors，错误处理库。
* expvar，expvar 包提供了公共变量的标准化接口，例如服务器中的操作计数器。 它通过 HTTP 以 JSON 格式在 /debug/vars 公开这些变量。
* image，2D图像库。
* log，日志相关库。
* math，数学相关。
* regexp，正则表达式。
* testing，测试相关库。
