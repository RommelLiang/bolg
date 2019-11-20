### Kotlin内联类、内部类、嵌套类
[kotlin内联类](https://blog.csdn.net/u013064109/article/details/84866092)
在开始介绍Kotlin内联类、内部类、嵌套类之前，先简要回顾一下java中的内部类
#### java中的内部类
`可以将一个类的定义放在另一个类的定义内部，这就是内部类`——《Java编程思想》
##### 概述
在java中，内部类有一中非常有用的特性，它允许将一些逻辑相关的类组织在一起，并控制位于内部的类的可见性。

* java内部类拥有其外围类所有元素的访问权，不需要特殊条件就可以访问外围类的所有成员。
* 内部类弥补了单继承的缺点。
* 匿名内部类可以让我们在使用回调函数时减少工作量（例如android中的点击事件）
* 内部类可以对同一包中的其他类隐藏起来(可以使用private和protected来修饰)
一个内部类的例子：
```
public class OutClass {
    private String outName;
    private class InnerClass{
        private String innerName;
        public InnerClass() {
            this.innerName = outName;
        }
    }
}
```
从代码中可以看出：
内部类可以直接访问外部类的成员变量，而内部类可以被private修饰。
##### 分类
在java中，内部类可分为四类：

* 成员内部类
* 方法内部类
* 匿名内部类
* 静态内部类
#### kotlin中的内部类
kotlin中的内部类与java中的内部类并没有什么不同，唯一的区别就是在没有任何修  饰的情况下，定义在另一个类内部的类将被默认为嵌套类，不持有外部类的引用。如果要声明一个内部类，必须显示的使用inner修饰符。
```
class OutClass {
    private val outName: String? = null

    private inner class InnerClass {
        private val innerName: String?

        init {
            this.innerName = outName
        }
    }
}
```
嵌套类和内部类在java和kotlin中的对应关系

| 类A在类B中声明 |java  | kotlin |
| --- | --- | --- |
| 嵌套类（不存储外部类的引用） |static class A|  class A|
| 内部类 （持有外部类的引用）| class A  | inner class A |
#### kotlin中的匿名内部类
kotlin总的匿名内部类在功能上与java一致，只是使用方法有些差别：
```
observer.setListener(object :Listener{
            override fun onStart() {
                
            }

            override fun onNext() {
                
            }

            override fun onEnd() {
                
            }
})
```
结合kotlin特性：**若函数最后一个参数为函数类型，该参数可以放到函数“()”的外面**
例如，android中的点击事件就可以简写为：
```
button.setOnClickListener {              AdapterActivity.launch(this)
}
```
