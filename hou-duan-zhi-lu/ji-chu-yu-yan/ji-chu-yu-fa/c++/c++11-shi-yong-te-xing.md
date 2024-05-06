# c++11实用特性

## 一.移动语义和右值引用

移动语义的本质是利用浅拷贝来代替反复进行的深拷贝，最常见的场景是函数返回string，如果不存在移动语义的话，并且编译器不进行RVO优化，string会被反复构造，有了移动语义，可以进行浅拷贝，效果如果把资源移动走了一样

想要理解移动语义，需要理解三个值和右值引用，可以看文章[C++右值引用与移动语义和完美转发_windsofchange的博客-CSDN博客_c++右值引用和移动语义](https://blog.csdn.net/ArtAndLife/article/details/120813556)

## 二.智能指针

智能指针是c++11中非常重要的一个知识，合理使用智能指针可以管理对象生命期，掌握shared\_ptr，unique\_ptr，weak\_ptr

构造shared\_ptr建议使用make\_shared，构造unique\_ptr建议使用make\_unique(c++14支持)

shared\_ptr需要注意循环引用的问题

## 三.lambda和function

lambda就是闭包，lambda和function结合可以达到强大的多态效果
