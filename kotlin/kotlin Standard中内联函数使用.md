#### let、with、run、apply、also、takeIf、takeUnless、repeat函数的使用

kotlin Standard.kt文件中，提供了一些扩展函数，这些扩展函数可以减少代码量，在使代码优美的同时，打打提高开发效率。它们分别为：
## run、with、let、also、apply 
##### let
let函数的定义如下：
`public inline fun <T, R> T.let(block: (T) -> R): R = block(this)`
默认当前这个对象作为闭包的it参数，函数接受一个lambda函数块返回值是函数里面最后一行，或者指定return
let函数的一般结构为：
       
```
obj.let {
    it.todo//it指代obj对象实例
    ...
}
//在需要判断obj是否为null时
obj?.let {
    it.todo//it指代obj对象实例
    ...
}
```
使用实例：初始化user
```
 val user = User()
 val result= user.let {
     it.account = "12306"
     it.address = "粤海街道"
     it.address
}
println(result)
//运行结果
>>粤海街道
```
适用场景：

* 用let函数处理需要针对一个可null的对象统一做判空处理
* 明确一个变量所在特定作用域的范围

##### with
with函数的定义如下：
`public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()`
with函数不是以扩展函数的形式存在，它是将对象作为参数，在函数块内通过this指代该对象。with函数是接收了两个参数，分别为对象receiver和一个lambda函数块，返回值为函数块的最后一行或指定return表达式。
with的一般结构为：
```
with(obj){
   this.todo
   todo//this可省略
   ...
}
```
使用实例：将地址影射到UI上
```
with(user){
 tView.text = address
}
```
适用范围：
适用于调用一个类的多个方法，可以省去对象名直接调用方法（例如将数据影射到ui上时）

##### run
run函数的定义如下：
`public inline fun <T, R> T.run(block: T.() -> R): R = block()`
run函数接受一个lambda函数块，以闭包的形式返回函数块的最后一行或指定return表达式。观察函数的定义可以发现，run函数为一个扩展函数，而其接受的参数和with函数第二个参数相同,run函数可以理解为let函数和with函数的结合体。
run函数的一般结构为：
```
obj.run {
   this.todo
   todo//this可省略
   ...
}
```
使用实例：将地址影射到UI上
```
user.run {
 tView.text = address
}
```
适用范围：
适用于let和run函数的场景，run函数相较于let函数省去了必须适用it指代参数的麻烦，相较于with函数弥补了对象判空的问题
##### also
also函数的定义如下：
`public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }`
also函数的定义和let函数的类似，只是also函数返回的为传入对象的本身。
also函数的一般结构和使用方法和let函数类似：
```
obj.also {
    it.todo//it指代obj对象实例
    ...
}
//在需要判断obj是否为null时
obj?.let {
    it.todo//it指代obj对象实例
    ...
}
```
适用范围：
also函数返回值微传入方法的对象本身，所以可用来进行函数的链式调用
##### apply
apply函数的定义如下：
`public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }`
apply函数的定义和run函数的类似，唯一的区别就是apply函数返回的为传入对象的本身。
```
apply函数一般结构如下：
obj.apply {
   this.todo
   todo//this可省略
   ...
}
```
使用实例：给对象赋值
```
var user = User().apply {
            account = "12306"
        }
```
适用场景：
apply函数和run函数除了返回值外，整体功能和作用类似，一般用于对象初始化时对属性进行赋值。
##### 总结：
这里我们总结对比一下这五个函数，这五个函数的特性非常简单，区别也无非是接受的参数和返回的类型不同。其中，对于with,run,apply接收者是this（可以省略），而let和also则使用it接受且不可s省略对于；
对于with,run,let返回值是返回值是函数里面最后一行，或者指定return，而apply和also返回值是调用者本身。

| 函数名 |  接受者|返回值  |
| --- | --- | --- |
| let | it |  最后一行|
| with| this |  最后一行|
| run|  this| 最后一行 |
| also| it |调用者本身 |
| apply| this| 调用者本身 |
如何选择可以参考下图

![](https://user-gold-cdn.xitu.io/2019/8/10/16c77b4552ca3d0e?w=1378&h=702&f=png&s=113797)
## takeIf、takeUnless、repeat

##### takeIf & takeUnless
takeIf函数的定义如下：
`
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? = if (predicate(this)) this else null
`
可以看出：takeIf函数接受一个<u>入参类型为调用者的类型T，返回值为Boolean类型的lambda函数块</u>。takeIf函数根据lambda函数返回值返回的数据，为true返回调用者本身，否则返回null

takeUnless函数的定义如下：
`public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? = if (!predicate(this)) this else null`
不难看出，takeUnless相对比takeIf只是在返回值调用predicate(this)进行了取反操作，takeUnless的作用效果和takeIf正好相反！
takeIf函数一般结构如下：
```
obj.takeIf{
    ...
    true/fals
}
```
使用实例：
```
//根据age为user赋值，若age在1-100之间，为user.age赋值age，否则user.age为null
var age
...
user.age = age.takeIf { 
   age in 1..99
}.toString()

user.age = age.ta { 
   age in 100..0
}.toString()
```
##### repeat
repeat函数的定义如下：
`public inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}`
函数接受一个int类型数据times，和一个入参为int类型，无返回值的lambda函数action，并通过for循环重复的调用times次action函数
函数的一般结构如下：
```
repeat(int){
    todo
}
```
使用实例：
```
//快速的为list添加十条数据
var list = ArrayList<User>()
repeat(10){
   list.add(User())
}
```
适用场景：该函数可以用来避免写循环语句，可以用来遍历数据。

## 结语：

Kotlin Standard.kt中的标准库函数已基本讲解完毕，其中涉及到了高阶函数和lambda函数，相关知识可通过[官方文档](https://www.kotlincn.net/docs/reference/)学习，同时建议读者将每个函数都实际敲一遍，并通过查看他们编译后的class文件加深对函数的理解。
