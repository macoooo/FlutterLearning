## 简介

新建一个Flutter工程，在main.dart中我们通常会看见这样的代码

```dart
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    final wordPair = new WordPair.random();
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.yellow,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text('Flutter demo')
        ),
        body: Center(
          child: Text(
              "center",
          style: TextStyle(
            fontSize: 40,
            fontWeight: FontWeight.bold,
            color: Color.fromRGBO(255, 222, 222, 0.5)
          ),
          ),
        ),
      ),
    );
  }
}
```

Widget这个词很是醒目，那么它到底是什么呢。做过原生开发的同学，第一反应就是类似原生的UI控件。那是不是，研究研究便知。

按住Command点一下，我们来看下Widget类的声明

```dart
// DiagnosticableTree 诊断树，提供调试信息
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key key;

  @protected
  // Flutter Framework在构建UI树时，会先调用此方法生成对应节点的Element对象。此方法是Flutter Framework隐式调用的，在我们开发过程中基本不会调用到。
  Element createElement();

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  // 静态方法，主要用于在Widget树重新build时复用旧的Widget
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

通常我们不会直接继承Widget实现自己的组件，而是通过继承`StatelessWidget`或`StatefulWidget`来间接继承`Widget`类来实现，下面我们来看下两种Widget内部工作原理

## StatelessWidget

无状态widget，通常这种Widget用于不需要维护状态的场景，也就是UI数据不需要更新

```dart
abstract class StatelessWidget extends Widget {
  /// Initializes [key] for subclasses.
  const StatelessWidget({ Key key }) : super(key: key);

  /// Creates a [StatelessElement] to manage this widget's location in the tree.
  ///
  /// It is uncommon for subclasses to override this method.
  @override
  StatelessElement createElement() => StatelessElement(this);

  @protected
  Widget build(BuildContext context);
}
```

`StatelessElement` 间接继承自`Element`类，与`StatelessWidget`相对应（作为其配置数据)

> context用来获取widget树上下文，可以找到当前widget的父级子级Widget
>
> BuildContext对象 其实就是Element对象。BuildContext接口是用来阻止对Element对象的直接访问。显而易见，Element是实现细节，系统不希望我们直接操作Element。

## StatefulWidget

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);

  @override
  StatefulElement createElement() => new StatefulElement(this);

  @protected
  State createState();
}
```

`StatefulElement` 间接继承自`Element`类，与StatefulWidget相对应（作为其配置数据）。`StatefulElement`中可能会多次调用`createState()`来创建状态(State)对象。

如果在不同地方多次插入同一个widget，每一个地方会创建一个`StatefulElement`和`State`，也就是一个`StatefulElement`会对应一个`State`

我们还可以看到一个差别就是没有了build方法，其实它是被放到了State类中

那我们来看看State到底是什么

```dart
abstract class State<T extends StatefulWidget> with Diagnosticable {
  T get widget => _widget;
  T _widget;

  BuildContext get context => _element;
  StatefulElement _element;
  bool get mounted => _element != null;

  @protected
  @mustCallSuper
  // 当widget第一次插入到widget树时会被调用
  void initState() {
    assert(_debugLifecycleState == _StateLifecycle.created);
  }
  
  @mustCallSuper
  @protected
  // 当Widget.canUpdate返回true时才会调用
  void didUpdateWidget(covariant T oldWidget) { }

  @protected
  @mustCallSuper
  // 为了开发调试提供的，在hot reload时会被调用，此回调在release模式下永远不会被调用
  void reassemble() { }

  @protected
  // 当状态数据发生变化时，调用这个方法来使用更新后的数据重建UI
  void setState(VoidCallback fn) {
    
  }

  @protected
  @mustCallSuper
  // 当State对象从树中移除时，会调用。如果包含此State对象的子树在树的一个位置移动到另一个位置时（可以通过GlobalKey来实现）。如果移除后没有重新插入到树中则紧接着会调用dispose()方法。
  void deactivate() { }

  @protected
  @mustCallSuper
  // 当State对象从树中被永久移除时调用；通常在此回调中释放资源。
  void dispose() {
    assert(_debugLifecycleState == _StateLifecycle.ready);
    assert(() {
      _debugLifecycleState = _StateLifecycle.defunct;
      return true;
    }());
  }

  @protected
  Widget build(BuildContext context);

  @protected
  @mustCallSuper
  // 当State对象的依赖发生变化时会被调用；例如：在之前build() 中包含了一个InheritedWidget，然后在之后的build() 中InheritedWidget发生了变化，那么此时InheritedWidget的子widget的didChangeDependencies()回调都会被调用
  void didChangeDependencies() { }
  
  //...还有一些debug信息
}
```

看到类中的声明，有许多关于**Widget生命周期**的接口

<img src="https://pcdn.flutterchina.club/imgs/3-2.jpg" alt="图3-2" style="zoom:70%;" />

上面是关于Widget层面的生命周期，在开发中App层面的生命周期也同样重要

## APP生命周期

在iOS开发中我们可能会在APP生命周期事件中做一些处理，比如App 从后台进入前台、从前台退到后台，或是ViewController生命周期。而在 Flutter 中，我们可以利用**WidgetsBindingObserver**类，来实现同样的需求。

```dart
abstract class WidgetsBindingObserver {
  // 页面 pop
  Future<bool> didPopRoute() => Future<bool>.value(false);
  // 页面 push
  Future<bool> didPushRoute(String route) => Future<bool>.value(false);
  // 系统窗口相关改变回调，如旋转
  void didChangeMetrics() { }
  // 文本缩放系数变化
  void didChangeTextScaleFactor() { }
  // 系统亮度变化
  void didChangePlatformBrightness() { }
  // 本地化语言变化
  void didChangeLocales(List<Locale> locale) { }
  //App 生命周期变化
  void didChangeAppLifecycleState(AppLifecycleState state) { }
  // 内存警告回调
  void didHaveMemoryPressure() { }
  //Accessibility 相关特性回调
  void didChangeAccessibilityFeatures() {}
}
```

我们注意到 【App生命周期变化】didChangeAppLifecycleState回调函数，他有一个参数AppLifecycleState，这个枚举类有三个值

- resumed：可见的，并能响应用户的输入。
- inactive：处在不活动状态，无法处理用户响应。
- paused：不可见并不能响应用户的输入，但是在后台继续活动中。

在这个回调中打印state，尝试看看前后台切换时App的State

```dart
class _MyHomePageState extends State<MyHomePage>  with WidgetsBindingObserver{
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) async {
    print("$state");
    if (state == AppLifecycleState.resumed) {
      //do sth
    }
  }
}
```

<img src="https://img.kancloud.cn/28/80/2880ffdbe3c5df3552c0b22c34157ae6_622x462.png" alt="图3-2" style="zoom:70%;" />

### 帧绘制回调

除了需要监听 App 的生命周期回调做相应的处理之外，有时候我们还需要在组件渲染之后做一些操作。

在 iOS 开发中，我们可以通过 dispatch_async(dispatch_get_main_queue(),^{…}) 方法，让操作在下一个 RunLoop 执行

其实，**在 Flutter 中实现同样的需求会更简单**：依然使用万能的 WidgetsBinding 来实现。

WidgetsBinding 提供了单次 Frame 绘制回调，以及实时 Frame 绘制回调两种机制，来分别满足不同的需求：

- 单次 Frame 绘制回调，通过 addPostFrameCallback 实现。它会在当前 Frame 绘制完成后进行进行回调，并且只会回调一次，如果要再次监听则需要再设置一次。

```dart
WidgetsBinding.instance.addPostFrameCallback((_){
    print(" 单次 Frame 绘制回调 ");// 只回调一次
  });
```

- 实时 Frame 绘制回调，则通过 addPersistentFrameCallback 实现。这个函数会在每次绘制 Frame 结束后进行回调，可以用做 FPS 监测。

```dart
WidgetsBinding.instance.addPersistentFrameCallback((_){
  print(" 实时 Frame 绘制回调 ");// 每帧都回调
});
```

上面讲述的这几种回调函数，我们一定要知道它们的作用，便于我们在开发中更好的运用，给予用户舒适的体验。