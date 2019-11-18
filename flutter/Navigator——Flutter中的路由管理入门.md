### 简介
路由(Route)在移动开发中通常指页面（Page），在Android中通常指一个Activity。路由管理，就是管理页面之间如何跳转，通常也可被称为导航管理。在Flutter中，不同页面之间进行切换和发送数据，这些页面被称为Route(路由),是由一个Navigator的小部件进行管理。Flutter中的路由管理和原生开发类似，导航管理都会维护一个栈，入栈(push)操作对应打开一个新页面，出栈(pop)操作对应页面关闭操作，而路由管理主要是指如何来管理路由栈。
### 一个简单的例子：
创建一个Flutter应用，启动之后点击按钮跳转一个新的页面。
```
class HomePage extends StatefulWidget {
  @override
  _HomePage createState() {
    // TODO: implement createState
    return _HomePage();
  }
}

class _HomePage extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Center(
      child: GestureDetector(
        onTap: () {
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) {
                return PageA();
              },
            ),
          );
        },
        child: Text("HomePage"),
      ),
    );
  }
}

class PageA extends StatefulWidget {
  @override
  _PageA createState() {
    // TODO: implement createState
    return _PageA();
  }
}

class _PageA extends State<PageA> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Center(
      child: Text("PageA"),
    );
  }
}
```
### 路由命名和路由表
要想使用命名路由，我们必须先提供并注册一个路由表（routing table），这样应用程序才知道哪个名字与哪个路由组件相对应。其实注册路由表就是给路由起名字，路由表的定义如下：
`Map<String, WidgetBuilder> routes;`
它是一个Map，key为路由的名字，是个字符串；value是个builder回调函数，用于生成相应的路由widget。我们在通过路由名字打开新路由时，应用会根据路由名字在路由表中查找到对应的WidgetBuilder回调函数，然后调用该回调函数生成路由widget并返回。
我们可以在`MaterialApp`中注册路由表：
```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
      routes: {
        "/home":(context)=>HomePage(),
        "/a":(context)=>PageA(),
        "/b":(context)=>PageB(),
        "/c":(context)=>PageC(),
      },
    );
  }
}
```
### Navigator中的方法
了解了路由的基本使用和跳转之后，我们来看一下Navigator中基本的方法：
![](https://user-gold-cdn.xitu.io/2019/9/12/16d25de2c55fd87c?w=1234&h=650&f=png&s=229700)

Flutter使用一个堆栈来管理路由，push为入栈，将元素添加到堆栈的顶部，相应的pop为出栈，同一堆栈中删除顶部元素。在Flutter总，push和pop，已经可以胜任我们大多数的业务场景了。push和pushNamed的基本使用我们已经在上文讲过了，Navigator的各个静态方法的使用基本和这两者类似。下面我们对其中的每个方法做一个简单的介绍：
#### pop & canPop
当我们要结束当前页面，返回上一页面时，我们可以使用pop让路由出栈：
`Navigator.pop(context);`
如果需要有返回参数给上一页面：
`Navigator.pop(context,"返回的数据");`
相应的，需要在修改跳转的：
```
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) {
      return PageA(
        name: "wall",
        age: "13",
      );
    },
  ),
).then((v) {
  //接收目标页面返回的参数
  setState(() {
    a = v;
  });
});
```
在then里接收第二个页面返回的参数，并使用setState刷新页面
canPop可以用来判断页面是否被弹出，根据官方的教程，初始化路由页面无法被pop，也就是说当栈内只有一个路由时，改方法会返回false，其他时刻返回true。
#### maybePop & popUntil
`maybePop`可以理解为canPop的升级版，canPop只用来判断页面是否可以被pop。而maybePop则对此进行了升级——如果可以pop则直接pop，否则什么都不做。
```
if (Navigator.canPop(context)) {
    Navigator.pop(context);
} else {
  //nothing
}
```
也就是说当前页面能返回就返回，返回不了就拉倒。
`popUntil`作用是反复执行pop 直到返回到我们指定的页面为止，popUntil接收一个函数，等到该函数的参数predicate返回true为结束pop操作：
`Navigator.popUntil(context,  ModalRoute.withName('/a'));`
假如原来的页面顺序为a->b->c->d,执行完上述代码后为：a。
![](https://user-gold-cdn.xitu.io/2019/9/16/16d360f2c7b90de2?w=760&h=656&f=png&s=17007)
该方法可以帮助我们清除栈内指定页面之上所有的页面，然后显示指定页面（例如快速回到主页）。
#### pushReplacement & pushReplacementNamed
`pushReplacement`和`pushReplacementNamed`的功能一致，它们二者的区别与`push`和`pushNamed`的区别一样——前者直接将页面入栈，后者通过路由命名的名字将页面入栈。这两对的使用方法也一致，不同的是`pushReplacement`和`pushReplacementNamed`不是讲新的页面直接入栈，而是替换掉栈顶的页面。类比Android原生可以理解为：在启动新的Activity时，finish()掉当前页面。
```
Navigator.of(context).pushReplacementNamed('/d');
```
![](https://user-gold-cdn.xitu.io/2019/9/16/16d37d1367a2ba77?w=1028&h=646&f=png&s=52717)
#### pushAndRemoveUntil & pushNamedAndRemoveUntil
同上，两者的区别不在赘述，它们的实现的功能为：向栈添加新的路由，并删除所有先前的路由，直到路由为指定的路由为止——例如在退出登录时我们可以直接清除栈内的页面返回首页。用法如下：
```
Navigator.pushNamedAndRemoveUntil(context, "/a",ModalRoute.withName("/a"));
```
假如原来的页面顺序为a->b->c->d,执行完上述代码后为：a->a。
![](https://user-gold-cdn.xitu.io/2019/9/16/16d360cf91d3831a?w=794&h=634&f=png&s=18597)
#### removeRoute & removeRouteBelow
`removeRoute`表示从Navigator中删除路由，同时执行Route.dispose释放Route自身资源，路由的生命周期结束。
`removeRouteBelow`从Navigator中删除路由，同时执行Route.dispose操作，要替换的路由是传入参数anchorRouter里面的路由。
#### replace & replaceRouteBelow
`replace`将Navigator中的路由替换成一个新路由。
`replaceRouteBelow` 将Navigator中的路由替换成一个新路由，要替换的路由是是传入参数anchorRouter里面的路由。
### 路由传参
普通路由中的传参如下所示：
```
Navigator.push(context,
    MaterialPageRoute(builder: (context) => Less(name: "new")));
//在新的Widget的构造方法里接受参数：
class Less extends StatelessWidget {
  final String name;

  Less({Key key, this.name}) : super(key: key){
    print("无状态组件$name:创建了");
  }
}
```
对于页面数据的回调，除了上文中讲的，还可以使用方法传递的方式实现：直接将父Widget的方法传递给子Widget，在新的Widget里调用传递过来的方法更新数据。
命名路由的传参有些许复杂：
```
//携带参数跳转
Navigator.pushNamed(context, "/a", arguments: {"name": "name"});
//在目标页面取值
Map arguments2 = ModalRoute.of(context).settings.arguments;
```
还有一种方案是在 onGenerateRoute 回调中利用URL参数自行处理。
路由跳转时写入参数：
```
onGenerateRoute: (RouteSettings settings) {
        WidgetBuilder builder;
        if (settings.name == '/a') {
          builder = (BuildContext context) => new Full();
        }
        return new MaterialPageRoute(builder: builder, settings: settings);
      },
```
### 总结
差不多絮叨完了，基本上只是方法罗列，进一步的探索和封转我也还在探索之中。最后，画图真难，谁有Mac下好的画图软件推荐啊。

![](https://user-gold-cdn.xitu.io/2019/9/16/16d383f725a4519c?w=180&h=180&f=png&s=33135)
Navigator——Flutter中的路由管理入门.md