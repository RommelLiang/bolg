### 前言
良好的编程风格的主要原则之一是避免代码中的任何重复。如果你用过Java（8之前）编写代码，很可能已经养成了什么东西都自己去实现的习惯。在Kotlin中，我们必须纠正这一习惯。
我们对集合的大部分任务都遵循几个通用的模式，kotlin通过lambda的帮助，很好的为集合提供了一个库。这就是今天的主角——kotlin中集合的函数式API，也就是kotlin标准库中和集合有关的一些函数。接下来我们就尝试使用一下几个常用的函数。
关于lambda表达式可以阅读[lambda表达式与Kotlin高阶函数](https://juejin.im/post/5d4eea0b6fb9a06aec262ea1)
### 基础：元素的过滤和变换
过滤和变换是集合操作的基础，许多集合操作都是借助它们实现的。其中过滤借助filter、变换借助map
#### filter
filter函数遍历集合并返回符合给定条件的元素：
```
val list = listOf(1,2,3,4)
val filter = list.filter { it == 2 }
println(list)
println(filter)
<<<[1, 2, 3, 4]
<<<[2]
```
filter返回了一个新的集合，集合中包含了符合给定条件的元素，移除了不想要的元素
#### map
map函数对集合中的每一个元素应用指定的函数并把结果收集到一个新的集合：
```
val list = listOf(1,2,3,4)
val map = list.map { it * 2 }
println(list)
println(map)
<<<[1, 2, 3, 4]
<<<[2, 4, 6, 8]
```
map返回了一个新的集合，其中每个元素相较于老集合翻倍了。

可以轻松的把多次这样的调用链接起来：（取出偶数并翻倍）：
```
val list = listOf(1,2,3,4)
list.filter { it % 2 == 0 }.map { it * 2 }
```
#### all、any & coun、find
all和any用来校验集合中多有元素是否都符合某个条件：
如果你要判断集合内所有的元素是否都满足某一条件，你可以使用all
```
val list = listOf(1,2,3,4)
println(list.all { it > 0 })//集合内的元素是否都大于0
println(list.all { it > 3 })//集合内的元素是否都大于3
>>>ture
>>>false
```
如果你要判断集合内是否至少有一个元素满足某一条件，你可以使用any
```
val list = listOf(1,2,3,4)
println(list.any { it > 4 })//集合内是否有元素大于4
println(list.any { it > 3 })//集合内是否有元素大于3
>>>false
>>>true
```
count用于统计集合内满足条件的元素，find负责找到第一个满足条件的元素
```
val list = listOf(1,2,3,4)
println(list.count { it > 4 })//集合内大于4的元素个数
println(list.count { it > 2 })//集合内大于2的元素个数
println(list.find { it > 2 })//集合内第一个大于2的元素
>>>0
>>>2
>>>3
```
#### groupBy
假设要将上文中的`list`数组按照奇偶华化成不同的分组，可以使用groupBy实现：
```
val list = listOf(1,2,3,4)
val groupBy = list.groupBy { it % 2 == 0 }
println(groupBy)
>>>{false=[1, 3], true=[2, 4]}
```
观察这个例子的输出，这次操作的结果是一个map，元素按照传入的条件进行分组。每一个分组都存储在一个列表中。
#### flatmap & flatten
flatmap函数可以做两件事情：（不得不想起RxJava中的操作符，详细文章writing中）

* 根据给定的函数对集合中的每个元素做变换
* 把多个列表合并成一个列表

通过一段实例可以更好理解这个概念：
```
val list = listOf(listOf(1, 2, 3), listOf(1, 2, 3))
println(list.flatMap { it.map { it * 2 } })
>>>[2, 4, 6, 2, 4, 6]
```
其流程如下：
首先将list中的每个元素扩大一倍，之后和并这两个数组合并为一个list。
接下来看一下flatten：
```
val list = listOf(listOf(1, 2, 3), listOf(1, 2, 3))
println(list.flatten())
>>>[1, 2, 3, 1, 2, 3]
```
如果你只需要合并而无需变换，可以直接使用flatten
### 序列
观察这段代码：
```
val list = listOf(listOf(1, 2, 3), listOf(1, 2, 3),listOf(1, 2, 3,4))
list.map { it.size }.filter { it > 3 }
```
通过上文的学习，我们可以轻松的知道这段代码的功能。但是，这个上述用到的函数都会创建一个中间集合，也就是说每一步结果都会被存在一个临时列表。当数据量庞大的时候，这种链式调用效率会很低。为了提高效率，可以把操作变成序列：
```
list.asSequence().map { it.size }.filter { it>2 }.toList()
```
这段代码的效果和前面的例子一模一样，但是这段代码没有穿件热河用于存储元素的中间集合。
接下来再看一段对比代码：
```
val list = listOf(listOf(1, 2, 3), listOf(1, 2, 3),listOf(1, 2, 3,4))
list.map {
    print("map:$it ")
    it.size
}.filter {
    print("filter:$it ")
    it>2
}
println()
list.asSequence().map {
    print("map:$it ")
    it.size
}.filter {
    print("filter:$it ")
    it>2
}.toList()
>>>map:[1, 2, 3] map:[1, 2, 3] map:[1, 2, 3, 4] filter:3 filter:3 filter:4 
>>>map:[1, 2, 3] filter:3 map:[1, 2, 3] filter:3 map:[1, 2, 3, 4] filter:4 
```
对比两段代码打印的数据。对序列来说，所有的操作都是按顺序执行在每一个元素上面，处理完上一个元素，紧接着处理第二个元素。
其中：map和filter被称之为中间操作，toList为末端操作。中间操作是惰性的，也即是：只有末端操作被执行时，中间操作才会被调用。
### 总结
kotlin中的集合操作就介绍到这里，有兴趣的可以去翻阅源码了解更多api。在kotlin中要纠正**一切都要自己去实现**的习惯


