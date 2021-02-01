## 状态管理

什么是状态管理？

在Flutter声明式的代码编写范式下，优雅的处理各个Widget状态

也就是我们怎么更优雅的做到数据更新UI同步更新。

### 基本分类

局部管理，官方也称 Ephemeral state，意思是短暂的状态，这种只有一个Widget内部使用

全局管理， 官方称 App state，即应用状态，非短暂状态。我们需要在好几个Widget之间共享的状态

### State

最简单的就是`setState`，其更新原理可以看看[这个](https://www.cnblogs.com/ahyang/p/12129450.html)

<img src="https://user-gold-cdn.xitu.io/2020/1/1/16f608e2ec41b049?w=2763&h=1065&f=png&s=275054" alt="img" style="zoom:80%;" />

#### 缺点

- 会刷新整个widget，Widget复杂的话会造成性能问题
- 数据和UI耦合，不利于测试和复用
- 无法跨组件传递数据

下面是几个Flutter中数据传递的方式

### 数据共享

有时我们会有场景几个界面共享数据，这时我们可以采用InheritedWidget组件

#### InheritedWidget

是Flutter的一个功能性Widget，它能有效地将数据在当前Widget中向它的子widget树传递，实现数据共享。即将InheritedWidget作为根Widget，子Widget来共享数据。自顶向下传递数据。感觉就像是一个共享数据的容器

#### 缺点

- 容易造成不必要的刷新(我们可以重写of方法优化刷新逻辑)
- 不支持跨页面(route)的状态，意思是跨树，如果不在一个树中，我们无法获取

#### Notification

通过WidgetNotificationListener 监听WidgetNotification数据变化，WidgetNotification内部通过Notification的dispatch（context）分发通知，向上传递数据（Notification中放进去数据）

#### EventBus

#### StreamBuilder



## Provider&Consumer

依赖注入与状态管理的结合

可以做到局部刷新

Provider将数据操作和UI放进一个里面了





