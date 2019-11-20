### lambda表达式与Kotlin高阶函数
https://www.kotlincn.net/docs/reference/lambdas.html（参考网址）

### 概念
lambda表达式，或者简称为lambda，本质上就是可以传递给其他函数的一小段代码。可以轻松地把通用代码结构抽取成库函数。
高阶函数就是以另一个函数最为参数或者返回值的函数。在kotlin中，函数可以用lambda或者函数引用来表示。因此，任何以lambda或者函数引用作为参数或返回值的函数都是高阶函数。

### lambda
java 8的新特性之一就是引入lambda表达式。函数式编程特供了一种解决问题的方法：**把函数当做值来对待**。可以直接传递函数，而不需要先声明一个类再传递这个类的实例（这种场景常出现在java匿名内部类实现接口回调）。高效且直接的传递代码块使得代码更加简洁。
我们来看一个例子，定义一个按钮的点击事件。
使用匿名内部类的方式实现：
```
button.setOnClickListener(new OnCLickListener(){
    @Override
    public void onCLick(View view){
        doSomething();
    }
});
```
而在kotlin和java 8中，可以使用lambda实现：
```
button.setOnClickListener{           
    doSomething()
}
```
这两段代码做的事情一模一样，但是后者更简洁易读。
#### lambda表达式的语法
一个lambda把一小段行为进行编码，可以将它当做一个值传递。可以被独立的声明并存储到一个变量中。
声明lambda表达式的语法如下：
```
{x:Int,y:Int -> x+y}
```
kotlin的lambda表达式始终用花括号包围，箭头把实参列表和lambda的函数体隔开。箭头前为参数，箭头后为函数体。
接下来我们看一下lambda的使用。
#### lambda表达式的使用
假设存在一个高阶函数，该函数接受一个**入参为两个Int类型的数据，返回值也为int类型的函数**，并返回一个int类型的数据，该数据为传入函数对2和3的某种运算结果，代码如下：
```
private fun lambdaFunction  (function:(Int, Int)->Int):Int{
        return function(2,3)
    }
```
如果我们要计算2和3的和，则可以这样调用：
```
lambdaFunction({x:Int,y:Int -> x+y})
```
但这段代码多少有点啰嗦，过多的符号破坏了代码的可读性，幸运的是和局部变量一样**如果lambda参数的类型可以被推导出来，无需显式地指定它**，那么代码可以精简为：
```
lambdaFunction({x,y -> x+y})
```
其次，**若lambda表达式是函数调动的最后一个实参，它可以放在花括号外面**，继续改进代码：
```
lambdaFunction(){x,y -> x+y}
```
最后，**当lambda是函数唯一的实参时，还可以去掉代码中的空括号对**，那么最后精简的代码为：
```
lambdaFunction{x,y -> x+y}
```
上述四种语法形式含义都是一样的，但最后一种更易读
同时，**如果当前上下文期望的是只有一个参数的lambda且这个参数的类型可以推导出来，则可以使用默认参数名称it指代命名参数**。例如
```
user.let{
    it.age
}
```
其中it指代入参View，注意**it约定能大大缩短代码，但不能滥用。尤其是在嵌套的情况下，最好显示的声明每个lambda参数**
接下来我们谈谈和lambda形影不离的概念：**从上下文中捕捉变量**
#### 在作用域中访问变量
首先，看一段java代码：
```
private void fun(User me) {
        int a = 100;
        new Runnable() {
            @Override
            public void run() {
                me.setAge(a);
            }
        };
    }
```
在函数内声明一个匿名内部类的时候，能够在这个匿名内部类中引用这个函数的参数和局部变量（**需要注意的是：在JDK8之前，如果我们在匿名内部类中需要访问局部变量，那么这个局部变量必须用final修饰符修饰**）。lambda表达式可以做同样的事情。我们使用forEach来展示这种行为：
```
private fun lambdaFunction  (list: List<User>,age: Int){
        list.forEach { 
            it.age = age
        }
    }
```
这种从lambda内部访问外部变量的行为，我们称这些变量被lambda捕捉。
至此，lambda表达式已基本介绍完毕，接下来介绍kotlin中的高阶函数：
### 高阶函数
回顾一下开篇讲的高阶函数定义：
>高阶函数就是以另一个函数最为参数或者返回值的函数。在kotlin中，函数可以用lambda或者函数引用来表示。因此，任何以lambda或者函数引用作为参数或返回值的函数都是高阶函数

通过定义可以看出，其实在前文中使用到的`lambdaFunction`便是一个高阶函数。在声明一个高阶函数之前，首先必须要知道什么是`函数类型`
#### 函数类型
回顾`lambdaFunction`函数：
```
private fun lambdaFunction  (function:(Int, Int)->Int):Int{
        return function(2,3)
    }
```
其中，`(Int, Int)->Int`声明了一个函数类型。函数类型包括参数类型和返回类型，使用->箭头讲参数类型和返回类型分隔。
需要注意的是：Unit类型用于表达函数不返回任何有用的值，在声明一个普通函数时可以省略，但是在一个函数类型的声明过程中总是需要一个显式的返回值，所以，在这种场景下Unit不可省略
```
(Int, Int)->Unit
```
同时，在lambda表达式`{x,y -> x+y}`中参数的类型是被省略的，因为它们的类型已经在函数类型的参数类型中指定了，所以无需在lambda本身定义中再次声明。
可以为函数类型声明中的参数指定名称：
`(a:Int, b:Int)->Int`，其对应的lambda表达式可以为：
```
{x,y -> x+y}
//或者
{a,b-> a+b}
```
参数名称不会影响类型的匹配，当声明一个lambda时，可以使用任意符合规则的名称，但命名会提升代码的可读性。
#### 入参为函数的函数
知道了声明一个高阶函数的方法之后，接下来就是去实现它。继续以lambdaFunction作为例子，它的代码如下：
```
private fun lambdaFunction  (function:(Int, Int)->Int):Int{
        return function(2,3)
}
```
使用方法为：
```
//计算2和3的乘级
lambdaFunction{x,y -> x*y}
//计算2和3的和
lambdaFunction{x,y -> x+y}
```

调用作为参数的函数和调用普通函数一样，如`lambdaFunction中调用function`
这其中的原理为：
>函数类型被声明为普通的接口：一个函数类型的变量是FunctionN接口的一个实现，每个接口定义了一个invoke方法没调用这个方法就会执行函数。一个函数类型的变量就是实现了对应FunctionN接口的实现类的实例，实现类的invoke方法包含了lambda函数体。

#### 返回值为函数的函数
从函数中返回一个函数并没有将函数作为参数进行传递那么有用。但在程序中的一段逻辑可能因为其他条件而发生变化，便是其大显身手的时机。依旧以乘法和加法举例：
```
private fun lambdaFunction  (a:Int):(a:Int,b: Int)->Int{
        if (a>0){
            return {a, b -> a*b}
        } else{
            return {a, b -> a+b}
        }
    }
lambdaFunction(2)(1,2)
val lam = lambdaFunction(0)
lam(1,2)
```
函数`lambdaFunction`根据入参a的值分别返回求和以及求积函数。其中，函数的返回值为`(a:Int,b: Int)->Int`,声明一个返回一个函数的函数，需要指定一个函数类型作为返回类型。

### 总结

至此，大家对Kotlin高阶函数与lambda表达式应该有了清晰的认识。但本文只作为高阶函数和lambda使用的入门教程，尚未解释lambda表达式会被编译成匿名类，以及由此引出的内联函数，也未涉及到高阶函数中的控制流和集合函数式API。欲了解相关知识，请持续关注作者博客。