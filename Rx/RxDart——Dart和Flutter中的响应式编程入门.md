### RxDart
今年年初开始尝试使用`Flutter`开发android APP，期间遇到了不少的坑，但总算是有惊无险。而在做Android原生开发时，`RxAndroid`让代码爽到飞起。为了找回那种熟悉的感觉，特意将`RxDart`引入到项目中。首先介绍一下`RxDart`是什么：
`RxDart`基于[ReactiveX](http://reactivex.io/)，由`Dart`标准库中`Stream`扩展而成，是一个为Dart语言提供响应式编程的库。
### 如何使用
#### 添加依赖
首先在`pubspec.yaml`中添加依赖：
```
dependencies:
  rxdart: ^0.22.1+1
```
#### 范例
接下来我们通过一段代码来看一下如何使用`RxDart`
```
Observable.just(1).listen(print);
>>>1
```

![](https://user-gold-cdn.xitu.io/2019/8/17/16c9ef1d22a15491?w=520&h=450&f=png&s=82844)
一行完事儿？这都是什么啊？别着急，这只是Dart语法的特性，接下来我们将上面的代码进行拆分：
```
//创建一个被观察者
var observable = Observable.just(1);
//创建一个观察者
var observa = (int num) {
    print(num);
};
//通过listen实现订阅
observable.listen(observa);
```
这下就明朗了，作为一个Android开发者，对这段代码充满亲切感。简直和RxJava一抹一样！！！
#### 创建被观察者
RxDart提供了很多方法供我们创建Observable，除了上文中的直接使用just工厂函数`从单个值中创建`，还有以下几种创建方法：

* 从一个Stream中创建：`Observable(Stream.fromIterable([1, 2, 3, 4, 5])).listen(print);`
* 创建周期性事件:`Observable.periodic(Duration(seconds: 1), (x) => x.toString()).listen(print);`这段代码每隔一秒钟打印一个整数并加一
* 从Future中创建：通过一个Future创建的Observable，会先等待Future执行完毕，完后发射数据，这个输出数据就是Future的执行结果，如果Future没有任何返回值，那么输出null。还可以`toStream`方法将Future转换为Stream然后再创建：
    
```
Observable.periodic(Duration(seconds: 1), (x) => x.toString()).listen(print);
Future<String> asyncFunction() async {
    return Future.delayed(const Duration(seconds: 10), () => "十秒后的数据");
}
Observable.fromFuture(asyncFunction()).listen(print);
```
代码输出如下：

![](https://user-gold-cdn.xitu.io/2019/8/17/16c9ef2e0f26a70b)
#### 操作符和数据变换
Rx最爽的是什么？当然是无处不在的操作符啦！我个人觉得操作符是Rx的灵魂，关于操作符原理大家可以阅读[从源码查看RxJava中的map和flatMap的用法和区别](https://juejin.im/post/5d5517526fb9a06af13d63a2)去了解。下面我们通过几个常用的操作符学习一下RxDart的操作符使用方法：
#### 过滤操作符
过滤操作符就是用来过滤掉Observable发射的一些数据，丢弃这些数据只保留过滤后的数据。
##### where
点了半天编辑器总是不自动补全Filter

![](https://user-gold-cdn.xitu.io/2019/8/17/16c9ef29f026f111?w=442&h=438&f=png&s=216254)
竟然没有Filter操作符号了，还好有个where
我们实现一个对奇偶进行过滤的操作：
```
Observable(Stream.fromIterable([1, 2, 3, 4, 5]))
      .where((num) => num % 2 == 0)
      .listen(print);
>>>2 4
```
##### skip
如果我们想跳过前三个数字，我们可以使用skip操作符：
```
Observable(Stream.fromIterable([1, 2, 3, 4, 5]))
      .skip(3)
      .listen(print);
>>> 4 5
```
##### take
和skip操作相反，如果我们去掉后面三个数据，只输出前面两个数据：
```
Observable(Stream.fromIterable([1, 2, 3, 4, 5]))
      .take(2)
      .listen(print);
<<<1 2
```
##### distinct
如果我们要过滤掉原始数据里重复的项：
```
Observable(Stream.fromIterable([1, 2, 2, 2, 3, 3, 4, 5]))
      .distinctUnique()
      .listen(print);
>>> 1 2 3 4 5
```
#### 变换操作符
过滤操作符就是用来过变换Observable发射的一些数据或者Observable本身，将被变换的对象转换成为我们想要的。
##### map
首先说一下`map`，依旧是实现一个基本的数据转换，将数字翻倍：
```
Observable(Stream.fromIterable([1, 2, 3, 4, 5]))
    .map((num) => num * 2)
    .listen(print);
<<<2 4 6 8 10
```
##### flatMap
接下来说一下`flatMap`，取出一个由数组组成的数组中的元素：
```
var list1 = [1, 2, 3];
var list2 = [4, 5, 6];
var list3 = [7, 8, 9];
var listAll = [list1, list2, list3];
Observable(Stream.fromIterable(listAll))
    .flatMap((listItem) => Observable(Stream.fromIterable(listItem)))
    .listen(print);
<<<1 2 3 4 5 6 7 8 9
```
##### concatMap
类似flatMap操作符，区别在于concatMap按次序连接而不是合并那些生成的Observables，然后产生自己的数据序列。
看看下面的代码：
```
var list1 = [1, 2, 3];
  var list2 = [4, 5, 6];
  var list3 = [7, 8, 9];
  var listAll = [list1, list2, list3];
  var changeConcatMap = (List<int> listItem) {
    print("concatMap开始变换了");
    return Observable(Stream.fromIterable(listItem));
  };
  var changeFlatMap = (List<int> listItem) {
    print("FlatMap开始变换了");
    return Observable(Stream.fromIterable(listItem));
  };
  Observable(Stream.fromIterable(listAll))
      .concatMap((listItem) => changeConcatMap(listItem))
      .listen(print);
  Observable(Stream.fromIterable(listAll))
      .flatMap((listItem) => changeFlatMap(listItem))
      .listen(print);
```
输出结果为：

![](https://user-gold-cdn.xitu.io/2019/8/17/16c9ef347a963ae8?w=810&h=370&f=png&s=77503)
#### 结合操作符
##### startWith
在数据序列的开头插入一条指定的项：
```
var observable = Observable(Stream.fromIterable([1, 2, 3, 4, 5]));
observable.startWith(9).listen(print);
>>>9 1 2 3 4 5
```
##### merge
使用Merge操作符你可以将多个Observables的输出合并，就好像它们是一个单个的Observable一样。但是可能会让合并的Observables发射的数据交错
```
Observable.merge([
    Stream.fromIterable([1, 2]),
    Stream.fromIterable([3, 4])
  ]).listen(print);
>>> 1 3 2 4
```
##### combineLatest 
当两个Observables中的任何一个发射了数据时，使用一个函数结合每个Observable发射的最近数据项，并且基于这个函数的结果发射数据。
```
var observable1 = Observable.just(1);
var observable2 = Observable.just(2);
Observable.combineLatest([observable1, observable2], (num) => num)
      .listen(print);
>>>[1,2]
```
##### mergeWith
mergeWith将多个被观察者发射的流合并成一个流。数据按照被发送的顺序传递
```
var observable1 = Observable.just(1);
  var observable2 = Observable.just(5);
  var observable = Observable(Stream.fromIterable([1, 2, 3, 4, 5]));
  observable
      .mergeWith([observable2,observable1])
      .listen(print);
>>> 1 5 1 2 3 4 5
```

### 结语
这只是一片RxDart使用的入门教程的。本文并未深入探讨RxDart的实现原理和逻辑，因为这些原理基本和RxJava中的类似。感兴趣的可以去关注我的RxJava系列的文章