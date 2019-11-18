### Flutter垂直和水平布局
所谓线性布局，即指沿水平或垂直方向排布子组件。Flutter中通过Row和Column来实现线性布局，类似于Android中的LinearLayout控件。
### 主轴和纵轴（MainAxis & CrossAxis）
对于线性布局，有主轴和纵轴之分，如果布局是沿水平方向，那么主轴就是指水平方向，而纵轴即垂直方向；如果布局沿垂直方向，那么主轴就是指垂直方向，而纵轴就是水平方向。在线性布局中，`MainAxisAlignment`和`CrossAxisAlignment`，分别代表主轴对齐和纵轴对齐。线性布局默认尽可能多的占用主轴空间。默认是`MainAxisSize.max`，效果类似于Android布局中的`match_parent`，而`MainAxisSize.min`表示尽可能少的占用水平空间，当子组件没有占满水平剩余空间，则Row的实际宽度等于所有子组件占用的的水平空间，效果类似于Android中的`wrap_content`；
需要注意的是：**Row中主轴方向为水平方向，Column中的主轴方向为垂直方向**，如图：
![9c50739bf3d71650418d361ac6df1b48.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p23)

Row和Column的基本使用和效果如下：
```
new Row(
    children: <Widget>[
        new View(color: Colors.red),
        new View(color: Colors.yellow),
        new View(color: Colors.blue,),
    ],
),

new Column(
    children: <Widget>[
        new View(color: Colors.red),
        new View(color: Colors.yellow),
        new View(color: Colors.blue,),
    ],
),
```
效果如图：
![d86ac24eefe7889f274d71c314c8f400.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p25)
其中Row的宽度和Column为填充父控件剩余所有空间。
#### 布局方向
`textDirection`表示子组件在水平方向的布局顺序(是从左往右还是从右往左)，默认为：`TextDirection.ltr`从左向右。在Column中使用此属性没有任何效果。对应效果分别为：
![43382f8f44fa00f60edec8e35a03c242.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p27)
verticalDirection：表示纵轴的对齐方向，默认是VerticalDirection.down，表示从上到下。在`Column`中效果如下
![1c59c27ab565bcd095c20ebe50a92679.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p34)
而在`Row`中则可以用改属性设置字组件头部对齐或者底部对齐，但一般采用`crossAxisAlignment`属性进行设置。
#### 对齐方式
#### MainAxisAlignment
`mainAxisAlignment`表示子组件在主轴方向上的对齐方式，如果`mainAxisSize`值为`MainAxisSize.min`，则此属性无意义，因为子组件的宽度等于Row的宽度。
对齐方式有`start、end、center、spaceBetween、spaceAround、spaceEvenly`。对应的效果分别为：
###### start和end
表示沿主轴初始方向，默认为这种效果，子组件布局方向沿着`textDirection`设定的布局方向，和`start`表示逆主轴初始方向，子组件布局方向逆着`textDirection`设定的布局方向，效果如图：
![f409cbecd791201a6e2d02286daff384.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p31)

##### center
`center`表示尽可能的将子组件靠近中间排列。效果如图：
![001ef248497114f666def8cead1cc4bd.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p29)

##### spaceBetween、spaceAround和spaceEvenly
`spaceBetween`表示在子组件之间均匀的添加空白区域，让每个组件和相邻的组件都有相同的间距，但首尾组件与父组件没有间隙。
`spaceAround`表示在子组件之间均匀的添加空白区域，让每个组件和相邻的组件都有相同的间距，但是首尾距离父边界的距离为间距的一半。
`spaceEvenly`表示在子组件之间均匀的添加空白区域，让每个组件和相邻的组件都有相同的间距，但是首尾距离父边界的距离和间距一样。
效果如图：
![d694fcf2fe8aa2cafa768d4323bb5a1f.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p33)
已`MainAxisAlignment.center`为例，其使用方法为：
```
new Row(
    mainAxisAlignment: MainAxisAlignment.center,
    children: <Widget>[
        new View(color: Colors.red),
        new View(color: Colors.yellow),
        new View(color: Colors.blue,),
    ],
),
```
#### CrossAxisAlignment
`CrossAxisAlignment`表示子组件在纵轴方向上的对齐方式。对齐方式有：`start、end、center、stretch、baseline`。
###### start和end
`start`和`end`是一对相反的对齐方式，在未设置`verticalDirection`属性时（默认`VerticalDirection.down`），`start`表示顶部对齐，`end`表示底部对齐。而当`verticalDirection`为`VerticalDirection.up`时，`start`表示底部对齐，`end`表示顶部对齐。效果如下图：
![b0e17fdf041ec97a2e075b15f9931fe7.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p35)
###### stretch和baseline
`stretch`表示子组件填满纵轴方向。
`baseline`表示沿横轴放置子级,以便其基线匹配，如果主轴是垂直的，那么它的效果和`start`一样。
### 特殊情况
如果`Row`里面嵌套`Row`，或者`Column`里面再嵌套`Column`，那么只有对最外面的`Row`或`Column`会占用尽可能大的空间，里面`Row`或`Column`所占用的空间为实际大小。
### 默认属性
通过阅读`Row`和`Column`的源码，可以了解到它们的默认属性：
```
Row({
    Key key,
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
  }) : super(
    children: children,
    key: key,
    direction: Axis.horizontal,
    mainAxisAlignment: mainAxisAlignment,
    mainAxisSize: mainAxisSize,
    crossAxisAlignment: crossAxisAlignment,
    textDirection: textDirection,
    verticalDirection: verticalDirection,
    textBaseline: textBaseline,
  );
  
  Column({
    Key key,
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
  }) : super(
    children: children,
    key: key,
    direction: Axis.vertical,
    mainAxisAlignment: mainAxisAlignment,
    mainAxisSize: mainAxisSize,
    crossAxisAlignment: crossAxisAlignment,
    textDirection: textDirection,
    verticalDirection: verticalDirection,
    textBaseline: textBaseline,
  );
```
概括如下：
| 属性 | 默认值 |
| --- | --- |
|  mainAxisAlignment| start |
|  mainAxisSize| max |
| crossAxisAlignment | center |
| verticalDirection | down |
### 总结
了解了Flutter线性布局之后，不难发现，其设计思想和使用方式都和React及其类似，建议将二者放在一起对比学习。
附上React相关知识链接：

[React Flexbox布局](https://reactnative.cn/docs/flexbox/)

[ CSS3 Flexbox 口诀](https://weibo.com/1712131295/CoRnElNkZ?ref=collection&type=comment#_rnd1565588114861)






