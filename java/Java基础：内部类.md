`可以将一个类的定义放在另一个类的定义内部，这就是内部类`——《Java编程思想》
在java中，内部类可分为四类：
1. 成员内部类
2. 方法内部类
3. 匿名内部类
4. 静态内部类

### 成员内部类
成员内部类是最普通的内部类:
```
public class OutClass {
    private String out;
    public String name = "out";

    public InnerClass getInner(){
        return new InnerClass();
    }
    public class InnerClass {
        private String inner;
        public String name = "inner";

        public String getOutName() {
            //访问外部类变量
            return OutClass.this.name;
        }

        public String getInnerName() {

            return name;
        }
    }
}

```
`InnerClass`就是一个内部类，它就像外部类`OutClass`的一个成员，可以无限制的访问外部类的成员变量和方法。
不过需要注意的是，当当成员内部类拥有和外部类同名的成员变量或者方法时（例如代码中的`name`），当通过内部类访问该变量时默认情况下访问的是成员内部类的成员。如果需要访问外部类的变量，则需要：`OutClass.this.name`。
外部类如果想要访问内部类的成员变量和函数，必须先创建一个成员内部类的对象，然后再通过该对象的引用进行访问。
如果要创建内部类，必须存在一个外部类的对象：
```
OutClass.InnerClass innerClass1 = outClass.getInner();
OutClass.InnerClass innerClass2 = outClass.new InnerClass();
```
和成员变量一样，内部类可以拥有private访问权限、protected访问权限、public修饰。
### 方法内部类
一个方法或者一个作用域里面的类，就是方法内部类。
```
private Object fun() {
        class Fun {

        }
        return new Fun();
    }
```
它的访问权限仅限于方法内，是不能有public、protected、private以及static修饰符的。
### 匿名内部类
作为Android开发者，接触的最多的内部类就是匿名内部类，没错，就在编写事件监听的代码时就是使用匿名内部类：
```
view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               
        });
```
大部分匿名内部类用于接口回调，匿名内部类也是不能有访问修饰符和static修饰符的。匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。
### 静态内部类
静态内部类也是定义在另一个类里面的类，如果把成员内部类理解为成员变量，那么久可以把静态内部类理解为静态成员变量。静态内部类不可以外部类的非static成员变量或者方法。
```
public class OutClass {
    private String out;
    public String name = "out";

    public InnerClass getInner(){
        return new InnerClass();
    }
    static class InnerClass {
        private String inner;
        public String name = "inner";
        
    }
}

//初始化
OutClass.InnerClass innerClass = new OutClass.InnerClass();
```
### 作用
总结内部类的作用主要有：

1. 内部类方法可以访问该类定义所在作用域中的数据，包括被 private修饰的私有数据，可以将将存在一定逻辑关系的类组织在一起，又可以对外界隐藏
2. 每个内部类都能独立的继承一个接口的实现，可以实现java单继承的缺陷，使得多继承的解决方案变得完整
3. 方便编写事件驱动程序和编写线程代码，当我们想要定义一个回调函数却不想写大量代码的时候我们可以选择使用匿名内部类来实现

需要注意的是：**非静态内部类和匿名内部类都会持有外部类的引用**。

![](https://user-gold-cdn.xitu.io/2019/11/21/16e8cb26baf634ed?w=365&h=388&f=png&s=157667)
等你发现内存泄露了你就知道我为什么要特别提醒一下了！

[相关文章推荐](https://juejin.im/post/5a903ef96fb9a063435ef0c8)
[Kotlin中的内部类](https://juejin.im/post/5d4e4dcae51d453b1d6482a7)