通过下标来获取和设置元素是处理集合最常见的操作之一，这篇文章就来学习一下kotlin中集合区间的确定。
### 通过下表来访问元素
在kotlin中，可以使用类似java中的方式来访问map中的元素，也就是使用方括号的方法：
```
var value = map[key]
map[key] = value
```
在kotlin中，下标运算是一个约定：使用下标运算符读取元素会被换位get，写入则为set。
### for循环的新写法：
回忆一下在java中写一个循环十次的for循环：
```
for(int i =0;i < 10;i++){
    todo()
}
```
而在kotlin中，我们可以这样做：
```
for (i in 0 .. 10){
    println(i)
}
>>>012345678910
```
kotlin中的in运算符用来检查某个对象是否属于集合。其中`..`用来创建一个区间，它使用的是`rangeTo`函数的一种简洁写法：`0 .. 10`等同于`0.rangeTo(10)`。kotlin标准库提供了了用于任意元素的rangeTo函数，前提是这些元素是可比较的（实现了Comparable接口的）。
#### until
再看另一种实现：
```
for (i in 0 until 10){
    println(i)
}
>>>0123456789
```
注意`..`和`until`的不同，前者为闭区间，后者为前闭后开。
#### downTo
假如我们要实现倒叙输出呢？
```
for (i in 10 .. 0){
    println(i)
}
```
上述代码不会产生任何输出，正确的做法为：
```
for (i in 10 downTo 0){
    println(i)
}
>>>109876543210
```
#### step
假如我们需要每隔n个元素再打印，可以使用step，它表示遍历的步长：
```
for (i in 0 .. 10 step 2){
    print(i)
}
>>>0246810
```
### 要下标还是元素
需要注意的是，上述代码中的`i`并代表`0 .. 10`中的具体数字，它只不表示集合中元素的下标。通过一段代码来阐述一下：
```
var abc = listOf("a","b","c")
for (i in abc){
    print(i)
}
>>>abc
```
#### indices
如果我们需要获取集合中元素的下标该如何呢？可以使用`indices`
```
var abc = listOf("a","b","c")
for (index in abc.indices){
    print("$index")
}
>>>012
```
#### withIndex
而如果下标和元素都需要呢？答案是`withIndex`函数
```
for ((index,value) in abc.withIndex()){
    print("$index:$value ")
}
>>>
0:a 1:b 2:c 
```

