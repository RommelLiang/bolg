## Flutter视图Widget生命周期
作为一个Android开发者，一定会对Activity的生命周期有这很深刻的印象，而当你在使用Flutter时，其中Widget就是View，其生命周期就是从View创建到销毁的过程。
Widget分为StatelessWidgetStatefulWidget 两种，这两种Widget的生命周期分别如下。
#### StatelessWidget的生命周期
无状态Widget的生命周期很简单，它只有一个生命周期：build
##### build
build函数用来构建视图，每次页面刷新是被调用，典型的用法如下：
```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Welcome to Flutter',
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text('Welcome to Flutter'),
        ),
        body: new Center(
          child: new Text('Hello World'),
        ),
      ),
    );
  }
}
```
#### StatefulWidget的生命周期
StatefulWidget生命周期整理如下图：
![4a6a0acf54ef661359da31579a7be6d9.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p18)
大致可分为三个阶段：
* 初始化：插入渲染树
* 运行中：在渲染树中存在
* 销毁：从渲染树中移除
##### 初始化阶段
###### createState
createState必须且仅执行一次，它用来创建state，当创建StatefulWidget时，该放方法被执行
###### initState
在创建StatefulWidget后，initState是第一个被调用的方法，同createState一样只被调用一次，此时widget的被添加至渲染树，mount的值会变为true，但并没有渲染。可以在该方法内做一些初始化操作。在在override时要低啊用super.initState()
```
@override
void initState() {
  super.initState();
  ...
}
```
###### didChangeDependencies
当widget第一次被创建时，didChangeDependencies紧跟着initState函数之后调用，在widget刷新时，该方法不会被调用。它会在“依赖”发生变化时被Flutter Framework调用，这个依赖是指widget是否使用父widget中InheritedWidget的数据。也即是只有在widget依赖的InheritedWidget发生变化之后，didChangeDependencies才会调用。
这种机制可以使子组件在所依赖的InheritedWidget变化时来更新自身！比如当主题、locale(语言)等发生变化时，依赖其的子widget的didChangeDependencies方法将会被调用。
```
//通过继承InheritedWidget，将当前计数器点击次数保存在ShareDataWidget的data属性中：
class ShareDataWidget extends InheritedWidget {
  ShareDataWidget({
    @required this.data,
    Widget child
  }) :super(child: child);

  final int data; //需要在子树中共享的数据，保存点击次数

  //定义一个便捷方法，方便子树中的widget获取共享数据  
  static ShareDataWidget of(BuildContext context) {
    return context.inheritFromWidgetOfExactType(ShareDataWidget);
  }

  //该回调决定当data发生变化时，是否通知子树中依赖data的Widget  
  @override
  bool updateShouldNotify(ShareDataWidget old) {
    //如果返回true，则子树中依赖(build函数中有调用)本widget
    //的子widget的`state.didChangeDependencies`会被调用
    return old.data != data;
  }
}

class _TestWidget extends StatefulWidget {
  @override
  __TestWidgetState createState() => new __TestWidgetState();
}
//然后我们实现一个子组件_TestWidget，在其build方法中引用ShareDataWidget中的数据。
class __TestWidgetState extends State<_TestWidget> {
  @override
  Widget build(BuildContext context) {
    //使用InheritedWidget中的共享数据
    return Text(ShareDataWidget
        .of(context)
        .data
        .toString());
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("Dependencies change");
  }
}

```
注意：**如果_TestWidget的build方法中没有使用ShareDataWidget的数据，那么它的didChangeDependencies()将不会被调用，因为它并没有依赖ShareDataWidget**。在依赖改变之后build方法也会被调用，所以在大多数场景下都无需使用didChangeDependencies。然而如果你需要在依赖改变后执行一些昂贵的操作，比如网络请求，这时最好的方式就是在此方法中执行，这样可以避免每次build()都执行这些昂贵操作。
##### build
build函数会在widget第一次创建时紧跟着didChangeDependencies方法之后和UI重新渲染是时调用。build只做widget的创建操作，如果在build里做其他操作，会影响UI的渲染效果
##### 运行中
###### didUpdateWidget
当组件的状态改变的时候就会调用didUpdateWidget,比如调用了setState.
##### 销毁
###### deactivate
当要将State对象从渲染树中移除的时候，就会调用 deactivate 生命周期，这标志着 StatefulWidget将要销毁。页面切换时，也会调用它，因为此时State在视图树中的位置发生了变化但是State不会被销毁，而是重新插入到渲染树中。
**重写的时候必须要调用 super.deactivate()**
###### dispose
从渲染树中移除view的时候调用，State会永久的从渲染树中移除，和initState正好相反mount值变味false。这时候就可以在dispose里做一些取消监听操作。
总结：

| 函数 | 调用次数 | 调用时间 |
| --- | --- | --- | 
| createState |  1|  第一次创建|
|  initState| 1 |  第一次创建|
|  didChangeDependencies| n |  第一次创建和依赖变化时|
|  build | n |第一次创建和UI重新渲染时|  
| didUpdateWidget | n |  第一次创建和UI重新渲染时|  
| deactivate | n | state对象将要移除时| 
|  dispose| 1 |  state对象移除|