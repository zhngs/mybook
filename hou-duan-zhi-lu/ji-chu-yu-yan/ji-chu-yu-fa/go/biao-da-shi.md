# 表达式

## 一.运算符

### 1.算术运算符

* \+, -, \*, /, ++, --

### 2.关系运算符

* \==, !=, <, >, <=, >=

### 3.逻辑运算符

* !, &&, ||

### 4.位运算符

* &, |, ^, <<, >>

### 5.赋值运算符

* \=, +=, -=, \*=, /=, %=
* <<=, >>=, &=, ^=, |=

### 6.其他运算符

* &：取地址运算符
* \*：取值运算符

## 二.控制流

### 1.if

*   if语句后判断条件不需要加括号

    ```go
    a := 3
    if a < 2 {
        //...
    } else if a < 5 {
        //...
    } else {
        //...
    }
    ```
*   if语句内可以定义局部变量

    ```go
    if a := 3; x > 2 {
        //...
    }
    ```

### 2.switch

*   switch可匹配多个条件，每条case下默认自带break，如果想要继续执行下一条case，可以使用fallthrough关键字

    ```go
    a := 1
    switch a {
        case 1, 2:
            //...
        case 3:
            //...
        default:
            //...
    }
    ```

### 3.for

*   go只有for一个循环语句，但对其做了功能拓展

    <pre class="language-go"><code class="lang-go">for i := 1; i &#x3C; 3; i++ {
        //...
    }
    ​
    for i &#x3C; 3 {
        //类似while(i &#x3C; 3)
    <strong>}
    </strong>​
    for {
        //类似while(true)
    }
    </code></pre>
*   for可以搭配range对数据进行迭代，对于字符串，数组，切片类型，返回下标和值；对于字典，返回键和值；对于通道，只返回一个值，即通道收到的值

    ```go
    data := []int{1, 2, 3}
    for i, v := range data {
        //...
    }
    ```

### 4.其他控制语句

* goto：直接跳转到某个位置
* break：终止switch，for，select语句
* continue：终止后续逻辑，直接进行下一轮循环
