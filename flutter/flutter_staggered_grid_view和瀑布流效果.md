### 前言
经一位前辈提点，问了一个在Flutter中如何实现瀑布流效果的问题。能力有限，自己实在无法徒手造轮子实现。只能求助github，万幸找到了一个开源的[flutter_staggered_grid_view](https://github.com/letsar/flutter_staggered_grid_view)。发现相关资料很少，写篇文章小小的解读一下。
### 配置
1. 添加依赖：
```
dependencies:
  flutter_staggered_grid_view: x.x.x
```
2. 安装
```
//在终端执行命令
flutter pub get
```
3. 导包
```
import 'package:flutter_staggered_grid_view/flutter_staggered_grid_view.dart';
```
之后使用`StaggeredGridView`这个Widget就可以实现了。
[Pub官网接入教程](https://pub.dev/packages/flutter_staggered_grid_view#-installing-tab-)
注意：如果遇到`Unhandled Exception: type 'SliverHitTestResult' is not a subtype of type 'BoxHitTestResult'`异常，请检查`flutter_staggered_grid_view`版本，并尝试升级。参考[#49](https://github.com/letsar/flutter_staggered_grid_view/issues/49)

### 使用
直接贴一段[flutter_staggered_grid_view](https://github.com/letsar/flutter_staggered_grid_view)上的代码（我给加点注释）：
```
new StaggeredGridView.countBuilder(
  crossAxisCount: 4,
  itemCount: 8,
  itemBuilder: (BuildContext context, int index) => new Container(
      color: Colors.green,
      child: new Center(
        child: new CircleAvatar(
          backgroundColor: Colors.white,
          child: new Text('$index'),
        ),
      )),
  staggeredTileBuilder: (int index) =>
      new StaggeredTile.count(2, index.isEven ? 2 : 1),
  mainAxisSpacing: 4.0,
  crossAxisSpacing: 4.0,
)
```
其效果如下：
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4df6a528b1b8a?w=986&h=1464&f=png&s=84794)
仔细看看这段代码，基本都好理解：里面放了八个子widget，主轴和纵轴方向的（[主轴和纵轴？](https://juejin.im/post/5d50fad5f265da039e12ae80)）间距都是4个像素；`staggeredTileBuilder`好像是根据奇偶数控制子VIew的大小。`crossAxisCount`就比较奇怪，按照字面意思是纵轴方向view的个数，代码里命名赋值为4，可跑起来怎么只有两列？
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4dfb3ea2904f7?w=256&h=254&f=png&s=52285)
### 粗读源码
首先看一下`StaggeredGridView`，它的构造方法有以下几种：
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4e088343bb20a?w=648&h=141&f=png&s=52735)
阅读官方的注释之后，我的理解为：
* 最常用的是`StaggeredGridView.count`和`StaggeredGridView.extent`，前者创建了一个在纵轴方向有固定数量子View的布局，后者类似，只是指定了纵轴方向View数量的最大值。但是这两个东西只适合用在子View数量较少且固定的场景下，他们都是通过一个`List<Widget>`接收固定数量的子布局。
* 如果要创建含有大量子布局的网格视图，就要使用`StaggeredGridView.builder`、`StaggeredGridView.countBuilder`或者
`StaggeredGridView.extentBuilder`。更高级的是自定义`SliverVariableSizeChildDelegate`一个，然后通过`StaggeredGridView.custom`创建布局。
前者的使用方法为：
以`StaggeredGridView.count`为例，其使用方法为：
```
StaggeredGridView
		.count(
          crossAxisCount: 4,
          mainAxisSpacing: 2,
          crossAxisSpacing: 2,
          children: <Widget>[
            item(1),
           	...
            item(13),
          ],
          staggeredTiles: const <StaggeredTile>[
            const StaggeredTile.count(2, 1),
           	...
            const StaggeredTile.count(2, 1),
            const StaggeredTile.count(2, 2),
            ...
            const StaggeredTile.count(2, 2),
          ],
        )
```
其效果为：
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4e20ceda33f46?w=297&h=613&f=png&s=21383)
先来分析一下上文中说到的问题——`crossAxisCount`。
翻看源码发现`crossAxisCount`在类`SliverStaggeredGridDelegateWithFixedCrossAxisCount`中，并有一条注释：`The number of children in the cross axis.`。根据注释还是感觉这个属性是用来指定纵轴方向Item的个数的，依然是无法解释上文中命名传入的是4却只有两列的问题。这时，我在作者提交的[README文件](https://github.com/letsar/flutter_staggered_grid_view)发现了下面一段话：
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4e30c027dd18e?w=1848&h=444&f=png&s=118849)
我理解的大致意思就是：`StaggeredGridView`需要知道每个子布局（tile）如何去显示，以及这些tile里是什么样的Widget。而这些`tile需要在轴上占有固定数量的单元格`。还有三种方式供我们去配置战友的单元格数量，一种是指定数量，一种是指定最大值范围，还有一张是动态的，由tile里widget自身决定。
注意`tile需要在轴上占有固定数量的单元格`。这句话，一切都明了了，`crossAxisCount`并不是设置的轴上有几个布局，而是将根布局（或者说屏幕）分为几个单元格，然后通过`staggeredTileBuilder`去指定每个布局占据几个单元格。而配置方式就是通过`StaggeredTile`的单个构造方法：
* `StaggeredTile.count`：固定数量
* `StaggeredTile.extent`：最大值范围
* `StaggeredTile.fit`：自适应内容
这样上面的问题就可以理解了，网格列数由两个属性决定，也可以加单粗暴的理解为：列数 = crossAxisCount / StaggeredTile指定的占据几个单元格。
### 瀑布流效果
对`flutter_staggered_grid_view`有了基本的了解之后，我们再去实现一个瀑布流：
```
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Container(
      color: Colors.white,
      child: new Container(
        width: 200,
        color: Colors.amberAccent,
        child: new StaggeredGridView.countBuilder(
          //滑动控制器
          controller: _scrollController,
          primary: false,
          //滑动方向
          scrollDirection: Axis.vertical,
          //纵轴方向被划分的个数
          crossAxisCount: 2,
          //item的数量
          itemCount: 12,
          /**
           * mainAxisSpacing:主轴item之间的距离（px）
           * crossAxisSpacing:纵轴item之间的距离（px）
           * */
          mainAxisSpacing: 2.0,
          crossAxisSpacing: 2.0,
          staggeredTileBuilder: (index) => StaggeredTile.fit(1),
          itemBuilder: (BuildContext context, int index) => new Container(
            color: Colors.green,
            //随机生成高度
            height: 100 + Random().nextInt(10) * 20.0,
            width: 20,
            child: new Center(
              child: new CircleAvatar(
                backgroundColor: Colors.white,
                child: new Text('$index'),
              ),
            ),
          ),
        ),
      ),
    );
  }
```
效果如下：
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4e425858d255b?w=301&h=610&f=png&s=18924)
终于有点瀑布流的样子了，
### 总结
德浅才疏，如有纰漏和不足，请大家指出，万分感谢
![](https://user-gold-cdn.xitu.io/2019/9/16/16d383f725a4519c?w=180&h=180&f=png&s=33135)