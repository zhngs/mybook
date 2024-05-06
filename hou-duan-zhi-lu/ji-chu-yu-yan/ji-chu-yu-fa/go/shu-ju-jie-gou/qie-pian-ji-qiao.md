# 切片技巧

> 参考[https://go.dev/wiki/SliceTricks](https://go.dev/wiki/SliceTricks)

### 一.增

#### **1.append vector**

```
a = append(a, b...)
```

#### **2.copy**

```
// 方式1
b := make([]T, len(a))
copy(b, a)
​
// 方式2
b = append([]T(nil), a...)
​
// 方式3
b = append(a[:0:0], a...)
​
// 方式4
b = append(make([]T, 0, len(a)), a...)
```

#### **3.expand**

```
// 在i位置拓展n个元素
a = append(a[:i], append(make([]T, n), a[i:]...)...)
```

#### **4.extend**

```
// 在尾部添加n个元素
a = append(a, make([]T, n)...)
​
// 扩展cap
if cap(a)-len(a) < n {
    a = append(make([]T, 0, len(a)+n), a...)
}
```

#### **5.insert**

```
// 插入单个值
a = append(a[:i], append([]T{x}, a[i:]...)...)
​
// 上面的方式新建了一个[]T{x}切片，下面的方式可以避免
s = append(s, 0 /* use the zero value of the element type */)
copy(s[i+1:], s[i:])
s[i] = x
```

#### **6.insert vector**

```
a = append(a[:i], append(b, a[i:]...)...)
```

#### **7.push**

```
a = append(a, x)
```

#### **8.push front/unshift**

```
a = append([]T{x}, a...)
```

### 二.删

#### **1.cut**

```
a = append(a[:i], a[j:]...)
​
// 注意如果元素的类型是指针或带有指针字段的结构体，上面这种方式并没有完美删除元素，下面是比较严谨的解决办法
copy(a[i:], a[j:])
for k, n := len(a)-j+i, len(a); k < n; k++ {
    a[k] = nil // or the zero value of T
}
a = a[:len(a)-j+i]
```

#### **2.delete**

```
// 方式一
a = append(a[:i], a[i+1:]...)
​
// 方式二
a = a[:i+copy(a[i:], a[i+1:])]
​
// 方式三，这种方式时间复杂度上更好，但是没有保证顺序
a[i] = a[len(a)-1] 
a = a[:len(a)-1]
```

#### **3.filter**

```
// 过滤出满足keep函数的值
n := 0
for _, x := range a {
    if keep(x) {
        a[n] = x
        n++
    }
}
a = a[:n]
​
// 方式2
b := a[:0]
for _, x := range a {
    if f(x) {
        b = append(b, x)
    }
}
```

#### **4.pop**

```
x, a = a[len(a)-1], a[:len(a)-1]
```

#### **5.pop front/shift**

```
x, a = a[0], a[1:]
```

### 三.改

#### **1.reversing**

```
for i := len(a)/2-1; i >= 0; i-- {
    opp := len(a)-1-i
    a[i], a[opp] = a[opp], a[i]
}
​
// 2行的实现，可读性更好
for left, right := 0, len(a)-1; left < right; left, right = left+1, right-1 {
    a[left], a[right] = a[right], a[left]
}
```

#### **2.deduplicate**

```
import "sort"
​
in := []int{3,2,1,4,3,2,1,4,1} // any item can be sorted
sort.Ints(in)
j := 0
for i := 1; i < len(in); i++ {
    if in[j] == in[i] {
        continue
    }
    j++
    // preserve the original data
    // in[i], in[j] = in[j], in[i]
    // only set what is required
    in[j] = in[i]
}
result := in[:j+1]
fmt.Println(result) // [1 2 3 4]
```

\
