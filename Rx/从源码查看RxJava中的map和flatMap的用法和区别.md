### 前言：
RxJava中提供了大量的操作符，这大大提高了了我们的开发效率。其中最基本的两个变换操作符就是`map`和`flatMap`。而其他变换操作符的原理基本与`map`类似。

* map和flatMap都是接受一个函数作为参数(Func1)并返回一个被观察者`Observable`
* Func1的< I,O >I,O模版分别为输入和输出值的类型，实现Func1的call方法对I类型进行处理后返回O类型数据，只是flatMap中执行的方法的返回类型为Observable类型

### 作用
`map`对Observable发射的每一项数据应用一个函数，执行变换操作。对原始的Observable发射的每一项数据应用一个你选择的函数，然后返回一个发射这些结果的Observable。

![](https://user-gold-cdn.xitu.io/2019/8/15/16c94634af12680b?w=1280&h=610&f=png&s=197431)
`flatMap`将一个发射数据的Observable变换为多个Observables，然后将它们发射的数据合并后放进一个单独的Observable。操作符使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后FlatMap合并这些Observables发射的数据，最后将合并后的结果当做它自己的数据序列发射

![](https://user-gold-cdn.xitu.io/2019/8/15/16c94636b957d21e?w=1280&h=620&f=png&s=160920)

### 使用方法：
通过代码来看一下两者的使用用方法：
#### map
```
Observable.just(new User("白瑞德"))
                  .map(new Function<User, String>() {

                      @Override
                      public String apply(User user) throws Throwable {

                          return user.getName();
                      }
                  })
                  .subscribe(new Consumer<String>() {

                      @Override
                      public void accept(String s) throws Throwable {

                          System.out.println(s);
                      }
                  });
<<<白瑞德
```
这段代码接受一个User对象，最后打印出User中的name。
#### flatMap
假设存在一个需求，图书馆要打印每个User借走每一本书的名字：
`User`结构如下：
```
class User {
    private String       name;
    private List<String> book;
}
```
我们来看一下`map`的实现方法：
```
Observable.fromIterable(userList)
                  .map(new Function<User, List<String>>() {

                      @Override
                      public List<String> apply(User user) throws Throwable {

                          return user.getBook();
                      }
                  })
                  .subscribe(new Consumer<List<String>>() {

                      @Override
                      public void accept(List<String> strings) throws Throwable {

                          for (String s : strings) {
                              System.out.println(s);
                          }
                      }
                  });
```
可以看到，`map`的转换总是一对一，只能单一转换。我们不得不借助循环进行打印。
下面我们来看一下`flatMap`的实现方式：
```
Observable.fromIterable(userList)
                  .flatMap(new Function<User, ObservableSource<String>>() {

                      @Override
                      public ObservableSource<String> apply(User user) throws Throwable {

                          return Observable.fromIterable(user.getBook());
                      }
                  })
                  .subscribe(new Consumer<String>() {

                      @Override
                      public void accept(String  o) throws Throwable {
                          System.out.println(o);
                      }
                  });
```
`flatmap`既可以单一转换也可以一对多/多对多转换。`flatMap`使用一个指定的函数对原始`Observable`发射的每一项数据之行相应的变换操作，这个函数返回一个本身也发射数据的`Observable`，因此可以再内部再次进行事件的分发。然后`flatMap`合并这些`Observables`发射的数据，最后将合并后的结果当做它自己的数据序列发射。
### 源码分析
下面我们就结合源码来分析一下这两个操作符。为了降低代码阅读难道，这里只保留核心代码：
#### map
```
public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
    //接受一个Function实例，并返回一个ObservableMap
    return new ObservableMap<T, R>(this, mapper);
}

public final class ObservableMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends U> function;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
        //调用用父类构造方法，初始化父类中的downstream
        super(source);
        this.function = function;
    }

    @Override
    public void subscribeActual(Observer<? super U> t) {
        source.subscribe(new MapObserver<T, U>(t, function));
    }

    static final class MapObserver<T, U> extends BasicFuseableObserver<T, U> {
        final Function<? super T, ? extends U> mapper;

        MapObserver(Observer<? super U> actual, Function<? super T, ? extends U> mapper) {
            super(actual);
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            v = mapper.apply(t);
            downstream.onNext(v);
        }
    }
}
```
这段代码是去掉map源码中一些校验和其它相关回调后的精简代码。接下来分析一下代码流程：

* 当在调用`map`时，map接受一个匿名内部类`Function`的实例，并返回一个`ObservableMap`对象。
* `ObservableMap`本质上是一个`Observable`，也是一个被观察者，其构造方法接受最外层的那个被`Observable`实例，和`Function`实例。`ObservableMap`重写了`subscribeActual`方法，在`subscribeActual`中使用新建了一个`MapObserver`实现了对原始`Observable`的观察。
* 原始的`Observable`中的数据变会被发送到`MapObserver`的实例中。
* `MapObserver`构造方法接收原始`Observable`的观察者`actual`，和`Function`的实例`mapper`
* `MapObserver`在其`onNext`方法中调用`mapper`的`apply`方法，获得该方法的返回值v
       apply方法就是map实例中：
        ```
        public String apply(User user) throws Throwable {
            return user.getName();
        }
        ```
* 调用`downstream`的onNext方法，并传入实参v。其中`downstream`是`MapObserver`父类中定义的变量，在`MapObserver`构造方法中`super(actual);`时初始化，其本身就是传入的`actual`，本质上就是最原始的`Observable`

整个流程可以概括如下：
存在一个原始的`ObservableA`和一个观察者`ObserverA`，当原始的被观察者`ObservableA`调用`map`,并传入一个匿名内部类实例化的’function‘，`map`新建并返回了一个被观察者`ObservableB`，通过`subscribe`让观察者`ObserverA`对其进行订阅。并重写`subscribeActual`方法，在其被订阅时创建一个新的观察者`ObserverB`其接受的，并用`ObserverB`对原始的`ObservableA`进行订阅观察。当原始的`ObservableA`发出事件，调用`ObserverB`的`onNext`方法，`subscribeActual`接受的观察者便是最原始的观察者`ObserverA`。`ObserverB`变执行通过匿名内部类实例化的’function‘的`apply`方法得到数据`v`，紧接着调用原始的`ObservableA`的`onNext`方法，并传入实参`v`，`ObserverA`观察到事件。
一句话概括：一个原始的被观察者和观察者，但是让原始的观察者去订阅一个新的观察者，当新的被观察者被订阅的时候，创建一个新的观察者去订阅原始的被观察者，并在监听的事件之后执行指定的操作后再通知原始观察者。

#### flatMap
`faltMap`和`map`的基本原理类似，其代码如下：
```
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper) {
        return new ObservableFlatMap<T, R>(this, mapper, delayErrors, maxConcurrency, bufferSize);
}


public final class ObservableFlatMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends ObservableSource<? extends U>> mapper;
    final boolean delayErrors;
    final int maxConcurrency;
    final int bufferSize;

    public ObservableFlatMap(ObservableSource<T> source,
            Function<? super T, ? extends ObservableSource<? extends U>> mapper,
            boolean delayErrors, int maxConcurrency, int bufferSize) {
        super(source);
    }

    @Override
    public void subscribeActual(Observer<? super U> t) {

        source.subscribe(new MergeObserver<T, U>(t, mapper, delayErrors, maxConcurrency, bufferSize));
    }

    static final class MergeObserver<T, U> extends AtomicInteger implements Disposable, Observer<T> {

        MergeObserver(Observer<? super U> actual, Function<? super T, ? extends ObservableSource<? extends U>> mapper,
                boolean delayErrors, int maxConcurrency, int bufferSize) {
            ...   
            this.observers = new AtomicReference<InnerObserver<?, ?>[]>(EMPTY);
        }

        @Override
        public void onSubscribe(Disposable d) {
            downstream.onSubscribe(this);
        }

        @Override
        public void onNext(T t) {
            
            ObservableSource<? extends U> p;
            p = mapper.apply(t);    
            subscribeInner(p);
        }

        @SuppressWarnings("unchecked")
        void subscribeInner(ObservableSource<? extends U> p) {
           
                InnerObserver<T, U> inner = new InnerObserver<T, U>(this, uniqueId++);
                p.subscribe(inner);
        }

        void drain() {    
            drainLoop();
        }
        void drainLoop() {
            final Observer<? super U> child = this.downstream;
            child.onNext(o);
        }
    }

    static final class InnerObserver<T, U> extends AtomicReference<Disposable>
    implements Observer<U> {

        private static final long serialVersionUID = -4606175640614850599L;
        final long id;
        final MergeObserver<T, U> parent;

        volatile boolean done;
        volatile SimpleQueue<U> queue;

        int fusionMode;

        InnerObserver(MergeObserver<T, U> parent, long id) {
            this.id = id;
            this.parent = parent;
        }

        @Override
        public void onNext(U t) {
            parent.drain();
        }
        
    }
}

```
上述代码即是`faltMap`精简后的源码，其中大部分代码的运作流程和前文中的`map`源码一致，我们继续延续上文中讲解中的观察者和被观察者。重点关注其不同的地方：
`faltMap`返回一个新的被观察者`ObservableB`，重写`ObservableB`的`subscribeActual`方法在原始的观察者`ObserverA`对其进行订阅时新建一个观察者`ObserverB`对原始的`ObservableA`进行订阅。新的观察者`ObserverB`持有原始的`ObserverA`和`faltMap`接收的匿名对象实例`function`。当`ObserverB`监听到原始的被观察者`ObservableA`的事件时，`ObserverB`调用`function`的`apply`方法获得新新的被观察者`ObservableC`，再创建一个新的观察者`ObserverC`对`ObservableC`进行订阅，`ObserverC`持有原始的观察者`ObserverA`，在`ObserverC`观察到被观察者`ObservableC`的时间时，调用原始的观察者`ObserverA`的方法。

### 结语
至此，map和flatMap已基本分析完毕，其中map的代码比较简单易懂，flatMap中还涉及到大量辅助操作，文中并未涉及到其中的合并等操作，阅读起来有些困难。如果仅仅是为了了解二者的原理，可以阅读`Single<T>`中的代码。其中的代码量远远少于`Observable`中的代码量。如果对RxJava基本的模式还不了解，可以阅读大神博客[手写极简版的Rxjava](https://github.com/OliverY/RxjavaYxj)