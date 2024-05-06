# context

### 1.简介

go中context分为两大类

* 可取消context，典型的是cancelCtx，以及继承cancelCtx的timerCtx
* 不可取消context
  * backgroundCtx，todoCtx一般用作最开始的ctx
  * withoutCancelCtx，没看到使用场景
  * valueCtx，用来存储一对key，value

### 2.原理

context最终组成的数据结构是一个树，指针的方向是叶子节点指向根节点

对于不可取消的context，无法知道自己的子节点

对于可取消的context，内部有一个存子节点的map，目的是自己取消的时候，通知到子节点
