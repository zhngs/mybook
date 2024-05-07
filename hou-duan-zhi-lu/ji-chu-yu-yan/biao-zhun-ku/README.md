# 标准库

go官方标准库：[https://pkg.go.dev/std](https://pkg.go.dev/std)

### 一.分类

* bufio，实现缓冲 I/O。 包装一个 io.Reader 或 io.Writer 对象，创建另一个对象（Reader 或 Writer），该对象也实现该接口，为文本 I/O 提供缓冲和一些其他帮助。
* builtin，内置包。提供一些内置类型，函数。
* bytes，字节切片相关函数，和strings包类似。
* cmp，和比较相关的包。
* compress，提供若干压缩算法。
* container，额外的容器，如heap、list、ring。
* context，协程之间控制退出或传值。
* crypto，密码库，提供多种算法。
* database，sql相关接口。
* encoding，编解码相关库。
* embed，提供对正在运行的 Go 程序中嵌入的文件的访问。
* errors，错误处理库。
* expvar，expvar 包提供了公共变量的标准化接口，例如服务器中的操作计数器。 它通过 HTTP 以 JSON 格式在 /debug/vars 公开这些变量。
* flag，命令行参数解析。
* fmt，类似于 C 语言的 printf 和 scanf 的函数来实现格式化 I/O。
* hash，哈希算法库。
* html，提供了转义和取消转义 HTML 文本的函数。
* image，2D图像库。
* io，提供io相关的基本接口。
* log，日志相关库。
* math，数学相关。
*
