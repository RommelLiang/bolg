### 前言
通过阅读[lambda表达式与Kotlin高阶函数](https://juejin.im/post/5d4eea0b6fb9a06aec262ea1)，你应该了解到在kotlin中传递lambda作为函数参数的语法与普通的表达式很相似。这篇文章则带你了解lambda的运作原理以及用来消除lambda带来的运行时开销的内联函数。
### 通过字节码分析lambda表达式
我们先申明一个高阶函数lambdaFunction，并使用lambda作为实参对齐进行低啊用：
```
object Lombda {
    @JvmStatic
    fun main(arg: Array<String>) {
        val lambdaFunction = lambdaFunction({ x, y -> x + y })
        println(lambdaFunction)
    }

    private fun lambdaFunction(function: (Int, Int) -> Int): Int {
        return function(2, 3)
    }
}
```
在Intellij IDEA中将其字节码文件反编译为java文件，代码如下：
```
public final class Lombda {
   public static final Lombda INSTANCE;

   @JvmStatic
   public static final void main(@NotNull String[] arg) {
      Intrinsics.checkParameterIsNotNull(arg, "arg");
      int lambdaFunction = INSTANCE.lambdaFunction((Function2)null.INSTANCE);
      System.out.println(lambdaFunction);
   }

   private final int lambdaFunction(Function2 function) {
      return ((Number)function.invoke(Integer.valueOf(2), Integer.valueOf(3))).intValue();
   }

   private Lombda() {
      INSTANCE = (Lombda)this;
   }

   static {
      new Lombda();
   }
}

```
点击Function2进入Functions.kt文件，部分代码截图如下：
![a597defe7148338bb1272f0ea26ae575.png](evernotecid://5331C73A-6F27-4874-A975-3426C7CF79E8/appyinxiangcom/25650301/ENResource/p20)
**在kotlin.jvm.functions包下Functions.kt文件中定义了一系列的接口，这些接口对应于不同参数数量的函数（in为函数的入参，out为函数的返回值）。每个接口定义了一个invoke方法没调用这个方法就会执行函数。一个函数类型的变量就是实现了对应FunctionN接口的实现类的实例，实现类的invoke方法包含了lambda函数体。一个lambda表达式就是一个FunctionN接口的实现。每个lambda表达式都会被编译一个匿名类（除非它是一个内联函数）。一旦实现，编译器就可以避免为每一个表达式都生成一个class文件。如果lambda表达式捕捉了变量，每个被捕捉的变量会在匿名类中有对应的字段。而且每次对lambda表达式的调用都会创建一个该匿名类新的实例。**
了解了lambda函数的实现细节，可以发现：每次调用一次lambda都会额外创建一个类，而且如果lambda捕捉了变量，每次调用都会创建一个新的对象。这回带来运行时的额外开销。接下来就由内联函数来解决这个问题
### 内联函数：消除lambda带来的运行时开销
>如果使用inline修饰符标记一个函数，在函数被使用的时候编译器并不会生成函数调用的代码，而是使用函数实现真实的代码替换每一次的函数调用。

也就是说：**函数体会被直接替换到函数被调用的地方，而不是正常的被调用**

#### 内联函数运作方法
首先我们定义一个内联函数，并对它进行调用
```
inline fun lambdaFunction(action:(Int,Int)->Unit){
        action(2,3)
 }   
 
fun main() {
        println("调用前")
        lambdaFunction { x, y -> print(x+y) }
        println("调用后")
    }
```
下面这段代码和`main()`的作用相同，并且会被编译为同样的字节码
```
fun _main_(){
        println("调用前")
        print(2+3)
        println("调用前")
    }
```
其中lambda表达式被内联了。由lambda表达式生成的自己码成为了函数调用者定义的一部分，而不是被包含在一个实现了函数接口的匿名类中。（我个人的理解为：抽取代码块，编译时将代码块填充进指定位置，即调用内联函数的位置）。函数体会被直接替换到函数被调用的地方。
如果在两个不同的位置使用同一个内联函数，并且使用的lambda不同，那么内联函数会在每一个被调用的位置被分别内联。内联函数的代码会被拷贝到使用它的位置，并把不同的lambda替换到其中。
#### 内联函数的限制
并不是所有的lambda函数都可以被内联。
**如果lambda在某个地方被保存起来，lambda表达式的代码将不能被内联。**
例如：
```
val la = {x:Int, y:Int -> print(x+y)}
lambdaFunction (la)
```
以函数类型的变量作为参数调用内联函数，这种情况下lambda并不会被内联。
需要注意的是：使用内联函数只能提高带有lambda参数的函数的性能。对于普通函数而言，JVM已经提供了强大的内联支持，在字节码中，每个函数的实现只会出现一次，并不需要跟内联函数一样，每个调用的地方都拷贝一次。而且，如果函数直接被调用，调用栈会更加清晰易读。
使用内联函数能给我们带来明显的好处：
避免运行时开销（节约函数调用，lambda创建匿名类和实例对象的开销）

### 结语
至此，kotlin中的lambda和内联函数已经基本分析完毕。kotlin标准库中提供了许多内联函数，以及操作集合的函数。这些函数可以大大减少代码量和工作效率。持续关注我的博客，了解更多kotlin标准库函数。