### 前言
RxJava不仅提供了大量的操作，例如`map`、`flatMap`（[相关博客](https://juejin.im/post/5d5517526fb9a06af13d63a2)），还支持自定义操作符。
官方文档给出了自定义操作符的相关建议：
如果操作符是用于Observable发射的单独的数据项，则使用序列化操作符`ObservableOperator`。
如果你的操作符是用于变换Observable发射的整个数据序列，则使用变换操作符`ObservableTransformer`。
### 自定义序列化操作符
最近项目中有了这样一个需求：从后台获取了原始数据DataA，在前端加工处理后成DataB提交给后台。项目中大量地方使用了这个方法，刚开始是使用了一个Utils进行数据转换。但是我发现项目中的网络请求都是使用RxJava实现的。那么RxJava中有没有什么优雅的实现数据转换呢？我首先想到的是`map`操作符，但是每次使用`map`还要传入转换逻辑，真的很麻烦。而序列化操作符干好可以完美的实现我们的要求。接下来我们就通过一个简单的例子来看如何自定义序列化操作符。
假设存在一个这样的`User`的实例，我们要从中获取到它的`name`
```
class User {
    private String name;
    private List<String> book;
}
```
这是个伪需求啊，我们完全可以使用`map`操作符实现这个功能：
```
map(new Function<User, String>() {
    @Override
    public String apply(User user) throws Throwable {
        return user.getName();
    }
})
```
但这中方法每次都要写一遍转换逻辑，也就是`return user.getName()`。如果转换逻辑很复杂呢，如果我们不想关心转换过程，但业务中却大量需要呢？接下来就是轮到自定义操作符出场了。
首先，我们自定义一个`ObservableOperator`：
```
public class UserToName<String>
        implements ObservableOperator<String, User> {
    @Override
    public Observer apply(Observer observer) {

        return new Observer<User>() {

            private Disposable mDisposable;

            @Override
            public void onSubscribe(Disposable d) {

                mDisposable = d;
                observer.onSubscribe(d);
            }

            @Override
            public void onNext(User o) {

                if (!mDisposable.isDisposed()) {
                    observer.onNext(o.getName());
                }
            }

            @Override
            public void onError(Throwable e) {

                if (!mDisposable.isDisposed()) {
                    observer.onNext(e);
                }
            }

            @Override
            public void onComplete() {

                if (!mDisposable.isDisposed()) {
                    observer.onComplete();
                }
            }
        };
    }
}
```
接下来使用`lift`操作符搭配使用它：
```
  User dostoyevsky = new User("陀思妥耶夫斯基", book1);
  Observable.just(dostoyevsky)
                  .lift(new UserToName<String>())
                  .subscribe(new Consumer<String>() {

                      @Override
                      public void accept(String s) throws Throwable {

                          System.out.println(s);
                      }
                  });
>>>陀思妥耶夫斯基
```
业务逻辑处只要一行代码就可以实现变换操作。
`UserToName`这个类实现了`ObservableOperator`接口，你也可以重写`UserToName`的构造方法做一些初始化操作。
`ObservableOperator`的源码如下：
```
public interface ObservableOperator<Downstream, Upstream> {
    @NonNull
    Observer<? super Upstream> apply(@NonNull Observer<? super Downstream> observer) throws Exception;
}
```
这段代码很简单，他做了一个`将一个方法用于原始的Observer并返回一个新的Observer。用新的Observer监听原始的Observable。而left也会返回一个新的Observable，该Observable发送String类型的数据，用原始的Observer去订阅它`。其中`Downstream`代表转换后的数据流，`Upstream`代表转换前的数据流。详细的原理参考[从源码查看RxJava中的map和flatMap的用法与区别](https://juejin.im/post/5d5517526fb9a06af13d63a2)

### 自定义变换操作符
开篇就说到序列化操作符只能对Observable发射的单独的数据项进行处理，也就是这能对那个`user`进行操作，但对这个Observable却无能无力，例如我们想要打印上述`user`中所有的书名`book`。和上文中的一样，第一个想到的操作肯定是使用`flatMap`：
```
flatMap(new Function<User, ObservableSource<?>>() {

                      @Override
                      public ObservableSource<?> apply(User user) throws Throwable {

                          return Observable.fromIterable(user.getBook());
                      }
                  })
```
但是我偏不，我就要自己去实现。

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99a7a0e4cce2b?w=340&h=342&f=png&s=60198)
首先实现一个`ObservableTransformer`
```
public class UserTransformer implements ObservableTransformer<User, String> {

    @Override
    public ObservableSource<String> apply(Observable<User> upstream) {

        return upstream.flatMap(new Function<User, ObservableSource<String>>() {

            @Override
            public ObservableSource<String> apply(User user) throws Throwable {

                return Observable.fromIterable(user.getBook());
            }
        });
    }
}
```
接下来使用`compose`操作符实现它：
```
Observable.just(dostoyevsky)
                  .compose(new UserTransformer())
                  .subscribe(new Consumer<String>() {

                      @Override
                      public void accept(String s) throws Throwable {
                          System.out.println(s);
                      }
                  });
```
我们看看`ObservableTransformer`的源码：
```
public interface ObservableTransformer<Upstream, Downstream> {
    @NonNull
    ObservableSource<Downstream> apply(@NonNull Observable<Upstream> upstream);
}
```
什么鬼玩意儿哦！这东西的上游数据类型标注位置和`ObservableOperator`刚好相反！！
* `ObservableOperator<Downstream, Upstream>`
* `ObservableTransformer<Upstream, Downstream>`
这些我都不在乎，看看它干了什么？
`将函数应用于上游原始的Observable并返回具有可选的不同元素类型的ObservableSource。`其中`Downstream`就是用来指定不同元素的类型。
### 二者的区别
看完代码相信已经基本了解了它们的区别了：

* 前者作用于用于观察者层面，只对原始的被观察者发出的数据进行变换
* 后者作用于整个序列，对被观察进行了变化。
### 总结
完了。其实本来想继续讲一下`lift`和`compose`的原理，接入看完源码后发现它们的原理也基本和`map`与`flatMap`一致，基本一模一样。想了解的可以看一下[从源码查看RxJava中的map和flatMap的用法和区别](https://juejin.im/post/5d5517526fb9a06af13d63a2)
求大神带我

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99a7d482dce99?w=610&h=470&f=png&s=134393)