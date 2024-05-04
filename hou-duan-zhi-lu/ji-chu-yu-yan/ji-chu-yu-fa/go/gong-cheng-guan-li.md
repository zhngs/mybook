# 工程管理

## 一.包管理

把相同的功能放到一个目录，就是一个包

go语言工程管理的基本单位是包，一个包给它的声明提供一个独立的命名空间

### 1.包声明

* 每一个go源文件开头都需要进行包声明，表示该源文件属于这个包
* 一个文件夹下可以有多个源文件，但是只能有一个包

### 2.包导入

*   使用import语句进行导入，有两种特殊情况需要注意：`重命名导入和空导入`。重命名导入的目的是为了解决包重名的问题；空导入是因为go语言如果导入的包的名字没有被引用，会产生编译错误，而有时候我们确实不会引用某个包，但是需要对包级别的变量执行初始化表达式求值，并执行init函数，这种情况就用到空导入

    ```go
    import (
        "crypto/rand"
        mrand "math/rand" //重命名导入
        _ "image/png" //空导入
    )
    ```
* note：当导入一个包时，编译器去哪获得这个包？首先会去`$GOROOT`下查找标准库，如果找不到，会向上寻找go.mod文件，根据其中的依赖进行导入包

### 3.包初始化

* 包初始化，首先初始化包级变量，然后如果包内有特殊函数init，再调用init函数，这时候一个包初始化完毕
* go语言的包依赖是一个有向无环图，加入包p导入了包q，那么可以保证包q会在包p之前初始化完成。并且因为是有无环图，包可以独立编译甚至并行编译，go工程的编译速度非常快

### 4.包可见性

* 包通过一个简单的规则来控制包内变量和声明的可见性：`导出的标识符以大写字母开头`

## 二.依赖管理

### 1.go环境变量

*   命令行执行go env可以查看go的环境变量，下面是几个比较重要的环境变量

    ```go
    $ go env
    GO111MODULE="on"    //开启go module模式
    GOPATH="/root/go"    //用来存储下载的源码和命令
    GOROOT="/usr/lib/golang"    //go发行版的根目录，其中提供了所有标准库的包
    GOPROXY="https://goproxy.cn,direct" //国内使用go需要设置代理，不然有些包无法下载
    ```

### 2.go module

* 自go1.13版本开始，go module成为默认依赖管理工具，取代了旧的GOPATH模式。当环境变量`GO111MODULE="on"`时go module被开启
* go module意味着`项目本身是一个模块，一个模块下可以包含多个包`
*   在go moudule模式下，项目根目录下有go.mod文件，内容形式大致如下

    ```go
    module example.com/mymodule
    ​
    go 1.14
    ​
    require (
        example.com/othermodule v1.2.3
        example.com/thismodule v1.2.3
        example.com/thatmodule v1.2.3
    )
    ​
    replace example.com/thatmodule => ../thatmodule
    exclude example.com/thismodule v1.3.0
    ```
* 和go.mod文件有一个相匹配的文件go.sum，同样是在项目根目录下，记录了项目依赖包的哈希值，如果依赖包的哈希值和go.sum里面的哈希值不匹配，go会拒绝构建，防止了依赖包被篡改的可能

### 3.go.mod语法

* go.mod里一共6个关键字：`module, go, require, replace, exclude, retract`，重点掌握module, go, require即可
* `module`：该关键字的语法是`module module-path`，意思就是表明该模块的路径
* `go`：关键字语法是`go minimum-go-version`，记录编译运行某项目的最小go版本
* `require`：关键字语法是`require module-path module-version`，用来记录项目依赖
* `replace`：关键字语法是`replace module-path [module-version] => replacement-path [replacement-version]`，用另一个模块版本或本地目录替换特定版本（或所有版本）的模块内容
* `exclude`：关键字语法是`exclude module-path module-version`，指定要从当前模块的依赖关系图中排除的模块或模块版本
* `retract`：一般用不到，主要用来撤回版本

### 4.版本号管理

* 版本号形式为`v1.4.0-beta.2`，1为主要版本号，4为次要版本号，0为补丁版本号，beta.2为预发布标识

| 版本阶段  | 例子                | 信息                                                                   |
| ----- | ----------------- | -------------------------------------------------------------------- |
| 开发中   | 伪版本号或者v0.x.x      | 表示模块正在开发中且不稳定，不提供向后兼容。伪版本号的写法类似于`v0.0.0-20170915032832-14c0d48ead0c` |
| 主要版本  | v**1**.x.x        | 表示向后不兼容的版本，每一个大版本的发布都不保证向后兼容之前的主要版本                                  |
| 次要版本  | vx.**4**.x        | 会更改模块的公共api，但不破坏调用代码，保证向后兼容性和稳定性                                     |
| 补丁版本  | vx.x.**1**        | 补丁版本不影响模块的公共api或者依赖，保证向后兼容                                           |
| 预发布版本 | vx.x.x-**beta.2** | 表示一个预发布里程碑，不提供稳定性保证                                                  |

* note：当主要版本>=2的时候，go.mod的module关键字的内容需要附加主要版本号。例如会从`module example.com/mymodule`修改为`module example.com/mymodule/v2`。这样做的原因是主要版本的发布不会向后兼容，提醒代码开发者显式修改go.mod文件中的依赖，对于依赖管理更加友好

## 三.go工具

### 1.go mod

*   go mod命令提供了对模块的操作能力

    ```go
    go mod download    # 下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录）
    go mod init        # 初始化当前文件夹, 创建go.mod文件
    ```

### 2. go test

* go test是对包的测试命令，在一个包中，以\_test.go结尾的文件不是go build命令的编译目标，而是go test的编译目标
* go test工具会扫描\*test.go文件来寻找特殊函数，并生成一个临时的main包来调用它们，然后编译和运行，并汇报结果，最后清空临时文件

## 四.创建一个go项目

*   首先创建一个目录，然后进入目录执行go mod init命令，一个go项目就创建完成

    ```go
    $ mkdir greetings
    $ cd greetings
    $ go mod init greetings # 这句命令创建了go.mod文件，并将模块命名为greetings
    go: creating new go.mod: module example.com/greetings
    ```
* 如果想要添加依赖包，可以在go.mod文件中添加依赖，然后执行go mod download；也可以使用go get packagename@v1.0命令，可以自动更新go.mod文件
