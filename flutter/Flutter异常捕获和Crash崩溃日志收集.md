### 没有公开的参考文献
[https://yrom.net/blog/2019/07/05/dump-crash-log-for-flutter/](https://yrom.net/blog/2019/07/05/dump-crash-log-for-flutter/)
### 前言
和Android中的Java语言类似，Dart中也可以通过try/catch/finally来捕获代码块异常。不同的是在Dart中发生异常的时候flutter APP并不会崩溃。在我的实践中，debug版中的Dart异常会表现为红屏加异常信息，而release版则是空白的白屏。下面我们就`从源码追溯Flutter的异常和捕获`
### Flutter捕获的异常
Flutter为我们提供了部分异常捕获。在flutter开发中大家肯定遇到过屏幕编程红色并带有错误信息的情况，甚至在Widget宽度越界时也会出现这样的错误提示界面。虽然代码出现了错误，但是并不会导致APP崩溃，Flutter会帮我们捕获异常。至于其中的原理，就需要我们去看一下源码了。
以`StatelessWidget`为例，`Widget`会创建对应的`Element`，（至于为什么要进入Element里面，请大家自行了解Widget的绘制原理）其代码如下：
```
abstract class StatelessWidget extends Widget {
  /// Initializes [key] for subclasses.
  const StatelessWidget({ Key key }) : super(key: key);

  /// Creates a [StatelessElement] to manage this widget's location in the tree.
  ///
  /// It is uncommon for subclasses to override this method.
  @override
  StatelessElement createElement() => StatelessElement(this);
  ....
}
```
追根溯源`StatelessElement`父类到`ComponentElement`，发现如下代码：
```
/// Calls the [StatelessWidget.build] method of the [StatelessWidget] object
/// (for stateless widgets) or the [State.build] method of the [State] object
/// (for stateful widgets) and then updates the widget tree.
@override
void performRebuild() {
 if (!kReleaseMode && debugProfileBuildsEnabled)
    ...
    try {
      built = build();
      debugWidgetBuilderValue(widget, built);
    } catch (e, stack) {
      built = ErrorWidget.builder(_debugReportException(ErrorDescription('building $this'), e, stack));
    } finally {
      // We delay marking the element as clean until after calling build() so
      // that attempts to markNeedsBuild() during build() will be ignored.
      _dirty = false;
      assert(_debugSetAllowIgnoredCallsToMarkNeedsBuild(false));
    }
    try {
      _child = updateChild(_child, built, slot);
      assert(_child != null);
    } catch (e, stack) {
      built = ErrorWidget.builder(_debugReportException(ErrorDescription('building $this'), e, stack));
      _child = updateChild(null, built, slot);
    }
}
```
根据注释不难发现，这个方法用来调用`StatelessWidget`或者`State`的`build`方法，并更新Widget树。不难发现，代码使用了`try/catch`包裹了`built = build();`也就是包裹了`build`执行的方法。当捕获到异常时使用`ErrorWidget`弹出错误提示。不难发现`ErrorWidget.builder`接受了一个`_debugReportException`方法返回的`FlutterErrorDetails`，并展示异常——这就是我们上文提到的红屏吧。进入`_debugReportException`方法内继续追踪我们需要的异常信息：
```
FlutterErrorDetails _debugReportException(
  DiagnosticsNode context,
  dynamic exception,
  StackTrace stack, {
  InformationCollector informationCollector,
}) {
  final FlutterErrorDetails details = FlutterErrorDetails(
    exception: exception,
    stack: stack,
    library: 'widgets library',
    context: context,
    informationCollector: informationCollector,
  );
  FlutterError.reportError(details);
  return details;
}
```
可以看到，异常信息通过`FlutterError.reportError`上报。进入该方法：
```
/// Calls [onError] with the given details, unless it is null.
static void reportError(FlutterErrorDetails details) {
    assert(details != null);
    assert(details.exception != null);
    if (onError != null)
      onError(details);
  }
```
最后调用了`onError`方法，追踪源码：
```
static FlutterExceptionHandler onError = dumpErrorToConsole;
```
`onError`是`FlutterError`的一个静态属性，它默认的处理方法是`dumpErrorToConsole`。如果我们更改异常的处理方式，提供一个自定的错误回调就可以了。
```
//在Flutter app入口配置自定义的异常上报回调(customerReport)
void main(){
  FlutterError.onError = (FlutterErrorDetails details) {
    customerReport(details);
  };
  runApp(MyApp());
}
```
在Flutter app入口配置自定义的异常上报回调`customerReport`，至于异常的上报，是在Flutter中直接请求网络还是与原生交互，我们暂不进行讨论。至此，我们就可以收集和处理那些Flutter为我们捕获的异常了。
### Flutter没有捕获的异常（并发异常）
虽然Flutter为我们在很多关键的方法进行了异常捕获，但遗憾的是，它并不能为我们捕获并发异常。同步异常可以通过try/catch捕获，而异步异常则不会被捕获：
```
try{
    Future.delayed(Duration(seconds: 1)).then((e) => Future.error("xxx"));
}catch (e){
    print(e)
}
```
上面的代码是捕获不了`Future`异常的。怎么办？
幸运的是，Flutter中与一个`Zone`的概念，Dart中可通过`Zone`表示指定代码执行的环境，不同的Zone代码上下文是不同的互不影响。类似一个沙盒概念，不同沙箱的之间是隔离的，沙箱可以捕获、拦截或修改一些代码行为，我们可以再这里面捕获所有没有被处理过的异常。
看一下`runZoned`的源码：
```
R runZoned<R>(R body(),{
    Map zoneValues,
    ZoneSpecification zoneSpecification, 
    Function onError
    }
)
```
* zoneValues: Zone 的私有数据，可以通过实例zone[key]获取，可以理解为每个“沙箱”的私有数据。
* zoneSpecification：Zone的一些配置，可以自定义一些代码行为，比如拦截日志输出行为等
* onError：Zone中未捕获异常处理回调，如果开发者提供了onError回调或者通

我们可以通过`onError`捕获异常就像下面这样：
```
runZoned(
      () {
        Future.error("error");
      },
      onError: (dynamic e, StackTrace stack) {
        reportError(e, stack);
      },
    );
```
我们可以让`runApp`运行在Zone中，这样就可以捕获我们Flutter应用中全部错误了！
```
runZoned(() {
    runApp(MyApp());
}, onError: (Object obj, StackTrace stack) {
    reportError(e, stack);
});
```
`zoneSpecification`则给了我们带来了更大的可能性，它提供了`forked`Zone的规范，使用在此类中给出的实力作为回调覆盖Zone中的默认行为。处理程序上具有相同命名方法的方法。例如：拦截应用中所有调用print输出日志的行为：
```
runZoned(
    () => runApp(MyApp()),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
            report(line)
      },
    ),
  );
```
这样使用`onError`和`zoneSpecification`搭配可以实现记录日志和手机崩溃信息：
```
void main() {
  runZoned(
    () => runApp(MyApp()),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
            report(line)
      },
    ),
    onError: (Object obj, StackTrace stack) {
      customerReport(e, stack);
    }
  );
}
```
### Flutter engine 异常捕获
flutter engine部分的异常，以Android为例，主要为libfutter.so发生的错误。这部分发生的异常的捕获方式和原生发生的异常的捕获方式一样，可以借用Bugly等实现。
### 总结
通过上述介绍，Flutter的异常捕获整理如下：
#### 异常收集
```
void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    customerReport(details);
  };
  runZoned(
    () => runApp(MyApp()),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
            report(line)
      },
    ),
    onError: (Object obj, StackTrace stack) {
      customerReport(e, stack);
    }
  );
}
```
#### 异常上报
其中异常的上报可以使用`MethodChannel`传递给Native，实现方式略，有兴趣的课参考：[Flutter与android之间的通讯](https://juejin.im/post/5d515337e51d4561f40add43)
而在我负责的项目中，我使用Bugly来收集异常信息，我们可以将native接收到的Flutter异常交给Bugly上报。
而收集到的崩溃信息则需要配置符号表(symbols)还原堆栈以确定崩溃信息。详情请参考：[构建系统加入Flutter符号表](https://blog.csdn.net/qq_30379689/article/details/90813052)




