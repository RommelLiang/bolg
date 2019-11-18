### 抛砖引玉
自从开始使用Flutter，接触最多的东西肯定少不了`StatefulWidget`和`StatelessWidget`。我本人在学习和了解它们的过程中也翻阅了大量的文档和资料，但发现他们都在讲二者的区别和使用场景以及案例——但是为什么要这么用呢？这是一个值得思考的问题。
### StatefulWidget和StatelessWidget简介
免不了俗，开篇也是先讲一下`StatefulWidget`和`StatelessWidget`的用法和区别吧。
Flutter中，一切皆Widget。Widget是视图的载体，而Widget包含两种，一种是不需要更改状态的Widget，`StatelessWidget`它没要需要管理的内部状态，是无状态的。另外一种是可变状态的，`StatefulWidget`它有需要管理的内部状态，使用`setState`来管理状态改变。
Widget是有状态的还是无状态的，取决于他们依赖于状态的变化：

* 有状态：交互或者数据改变导致Widget改变，例如改变文案
* 无状态：不会被改变的Widget，例如纯展示页面，数据也不会改变

#### 特别提示
我特意用一个标题来吸引大家注意，是因为我在好几篇博客看到了类似下面的话：
>Flutter 里面包含两种widget，一种是不可变的Widget——StatelessWidget，另外一种是可变的Widget——`StatefulWidget`

**这是大错特错的！！！**，因为Widget只是视图的“配置信息”，是数据的映射， **`Widget`是不可变的，不可变的！！**。变的只是Widge里面的状态，也就是State。
贴一段`Widget`源码的截图
![](https://user-gold-cdn.xitu.io/2019/9/12/16d23d8b89d9c308?w=643&h=86&f=png&s=15438)
注意其中的`“A widget is an immutable description of part of a user interface”`。Widget只是用户界面一部分不可变的描述——至于为什么不可变以及都不可变了还怎么刷新UI，这两个问题接下来我会用一片博客详细介绍一下Flutter的渲染机制。

#### StatelessWidget
StatelessWidget是一个没有状态的widget——没有要管理的内部状态。它通过构建一系列其他小部件来更加具体地描述用户界面，从而描述用户界面的一部分。当我们的页面不依赖Widget对象本身中的配置信息以及BuildContext时，就可以用到无状态组件。例如当我们只需要显示一段文字时。实际上Icon、Divider、Dialog、Text等都是StatelessWidget的子类。
`StatelessWidget`的基本使用如下：
```
class Less extends StatelessWidget {
  final String text;

  const Less({Key key, this.text}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Text(text);
  }
}
```
`Less`包含了一个从外部接受一个不可变的数据源text并将它显示。
无状态的组件的声明周期只有一个：`build`，它只会在三种情况下被调用:

* 将widget插入树中的时候，也就是第一次构建
* 当widget的父级更改了其配置时，例如，Less的父类改变了text的值
* 当它依赖的InheritedWidget发生变化时

#### StatefulWidget
StatefulWidget是可变状态的widget。使用setState方法管理StatefulWidget的状态的改变。调用setState通知Flutter框架某个状态发生了变化，Flutter会重新运行build方法，应用程序变可以显示最新的状态。
状态是在构建widget的时候，widget可以同步读取的信息，而这些状态会发生变化。要确保在状态改变的时候即使通知widget进行动态更改，就需要用到`StatefulWidget`。例如一个计数器，我们点击按钮就要让数字加一。在Flutter中，Checkbox、FadeImage等都是有状态组件。
`StatefulWidget`的基本使用如下：
```
class Full extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _Full();
  }
}

class _Full extends State<Full> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new GestureDetector(
      onTap: onClick,
      child: new Text("$count"),
    );
  }

  void onClick() {
    setState(() {
      count += 1;
    });
  }
}
```
`Full`包含了一个内部持有的int状态，每次点击自增一，平使用setState刷新页面显示最新的值。
`StatefulWidget`的生命周期比较复杂，有兴趣的可以去看我的另一篇博客：[Flutter视图Widget生命周期](https://juejin.im/post/5d4e5f016fb9a06b1a567a16)

### StatefulWidget和StatelessWidget的实用场景
在涉及到Widget的工作时，遇到的头等大事就是确定widget应该使用StatefulWidget还是StatelessWidget？
简单的说，如果不需要自己维持状态就使用`StatelessWidget`，否则使用`StatefulWidget`。
进一步分析，根据上文的介绍，我们不难发现：

* 如果用户交互或数据改变导致widget改变，那么它就是有状态的。
* 如果一个widget是最终的或不可变的，那么它就是无状态。

而状态的管理可能有三种方式——自己管理，父widget管理以及两者混合搭配。我们可以参考下面的规则选择Widget：

* 如果widget的状态取决于动作，那么最好是由widget自身来管理状态，也就是使用`StatefulWidget`，例如动画；
* 如果状态是用户数据，则最好用父widget管理，也即是使用`StatelessWidget`，例如一个列表单个Item的选中状态；
* 如果还是摇摆不定，别问，问就是`StatelessWidget`。

### 更进一步的思考
前面我们已经知道了，Widget是不可变的，如果要改变就要重新创建。而StatefulWidget使用State来通过控制自身状态来为自己标记状态，这样就可以在下一次系统重绘检查时重新创建。
#### 为什么要选用StatelessWidget
通过上面的介绍，大家不难发现StatefulWidget几乎是这样一个存在——我在任何需求下使用它都能实现想要的效果，那么我们为什么不一股脑全部使用它呢？既然它也能实现StatelessWidget的效果，那我们还要StatelessWidget做什么？StatefulWidget就是一个全能的存在啊！！
为了解释这个疑问，我们就要去了解一下StatefulWidget伴随着全能而来的代价！
首先我们粗略的追溯一下setState的刷新源码：
```
@protected
void setState(VoidCallback fn) {
    ...
    _element.markNeedsBuild();
}

void markNeedsBuild() {
    ...
    if (dirty)
      return;
    _dirty = true;
    owner.scheduleBuildFor(this);
}

void scheduleBuildFor(Element element) {
    ...
    if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
      _scheduledFlushDirtyElements = true;
      onBuildScheduled();
    }
    _dirtyElements.add(element);
    element._inDirtyList = true;
    ...
}
```
去掉所有的assert校验，只保留关键代码，我们发现setState会调用`element`的`markNeedsBuild`方法，用来标记当前`element`为dirty状态，也就是需要build。并执行`BuildOwner`的`scheduleBuildFor`方法，`BuildOwner`是负责管理`element`的。直接追溯到`onBuildScheduled`，该发放的实现为`widget/binding.dart`中的`_handleBuildScheduled`方法，其中调用了`scheduler/binding.dart`中的`ensureVisualUpdate`，最后调用了`scheduleFrame`方法：
```
void scheduleFrame() {
    if (_hasScheduledFrame || !_framesEnabled)
      return;
    assert(() {
      if (debugPrintScheduleFrameStacks)
        debugPrintStack(label: 'scheduleFrame() called. Current phase is $schedulerPhase.');
      return true;
    }());
    window.scheduleFrame();
    _hasScheduledFrame = true;
  }
```
关键的代码出现了：`window.scheduleFrame()`
这是一个Native方法：
![](https://user-gold-cdn.xitu.io/2019/9/12/16d2452f982a3f75?w=661&h=184&f=png&s=30862)

实际上setState只是用来标记state对象需要根据已经变更的状态重新build来创建新的widget。调用setState将会出发每个子Widget的构造方法以及build方法。这意味着如果根布局是一个`StatefulWidget`，那么setState之后，整个页面所有的widget都会重建。
通过代码来验证一下：
```
class FulBackPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _FulBackPage();
  }
}

class _FulBackPage extends State<FulBackPage> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Column(
      children: <Widget>[
        Full(name: "A"),
        Full(name: "B"),
        Full(name: "C"),
        Full(name: "D"),
        Less(name: "E"),
        GestureDetector(
          onTap: () {
            setState(() {});
          },
          child: Text("点击"),
        )
      ],
    );
  }
}

class Full extends StatefulWidget {
  final String name;

  Full({Key key, this.name}) : super(key: key) {
    print("有状态组件$name:创建了");
  }

  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _Full();
  }
}

class _Full extends State<Full> {
  @override
  Widget build(BuildContext context) {
    print("有状态组件${widget.name}:build了");
    return new GestureDetector(
      onTap: () {
        setState(() {});
      },
      child: new Text(widget.name),
    );
  }
}

class Less extends StatelessWidget {
  final String name;

  Less({Key key, this.name}) : super(key: key){
    print("无状态组件$name:创建了");
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    print("无状态组件$name:build了");
    return new Text(name);
  }
}
```
每次点击`FulBackPage`的按钮刷新页面，日志输出如下：
![](https://user-gold-cdn.xitu.io/2019/9/12/16d247af0100a050?w=234&h=189&f=png&s=14155)
哈哈哈，发现了一个StatelessWidget的用处了。因为StatelessWidget没有setState方法，所有它可以强制的减少开发者滥用setState，导致过多的页面被刷新。官方也是推荐首选使用StatelessWidget，也就是说要减少页面刷新的区域和层级！！！
好失望了，难道设计出StatelessWidget就是为了规范开发者的行为吗？？？？

![](https://user-gold-cdn.xitu.io/2019/9/12/16d2480d73dada69?w=225&h=225&f=png&s=105463)

我编不下去了啊！！！翻了翻两个widget的build源码，除了一个多了个state之外，我也没发现什么端倪。
总之：

* 优先使用StatelessWidget
* 含有大量子Widget（如根布局、次根布局）最好使用StatelessWidget
* StatefulWidget最好用在子节点，同时尽量减少它的子节点。

### 总结
开篇抛出的问题我还是没有彻底想明白：

![](https://user-gold-cdn.xitu.io/2019/9/12/16d249214497c879?w=180&h=180&f=png&s=31309)

