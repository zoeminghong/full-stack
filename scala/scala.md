在Scala中，一切都是对象，包括数值类型

scala比java的优势？

- 更灵活的泛型
- 简洁、优雅、灵活的语法
- 强拓展性

SBT

包含Scala编译器、标准库、第三方资源

单例对象

无法通过new方式创建对象

```scala
object OptionDemo {
...
}
```

隐式参数

只接受单一参数，且函数体中只调用一次，可以使用_代替参数

```scala
(s:String)=>s.toUpperCase()
可以改写为
_.toUpperCase()
```

构造函数

```
class Json(s:String){

}
```

这里的()中的就是构造函数，构造函数值相同，则对象也就相同

伴生类与伴生对象

单例对象与类同名时，这个单例对象被称为这个类的伴生对象，而这个类被称为这个单例对象的伴生类。伴生类和伴生对象要在同一个源文件中定义，伴生对象和伴生类可以互相访问其私有成员。不与伴生类同名的单例对象称为孤立对象。

```scala
class ChecksumAccumulator {
  private var sum = 0
  def add(b: Byte) {
    sum += b
  }
  def checksum(): Int = ~(sum & 0xFF) + 1
}
 
object ChecksumAccumulator {
  private val cache = Map[String, Int]()
  def calculate(s: String): Int =
    if (cache.contains(s))
    cache(s)
  else {
      val acc = new ChecksumAccumulator
      for (c <- s)
        acc.add(c.toByte)
      val cs = acc.checksum()
      cache += (s -> cs)
      println("s:"+s+" cs:"+cs)
      cs
    }
 
  def main(args: Array[String]) {
    println("Java 1:"+calculate("Java"))
    println("Java 2:"+calculate("Java"))
    println("Scala :"+calculate("Scala"))
  }
}
```

Unit

Unit对象存在副作用，无法知道函数的执行结果，有返回值的函数属于无副作用函数

高阶函数

假如某函数接受其他函数参数并返回函数，成为高阶函数

Any类型

Any类是所有类的根类

偏移函数

```scala
val another_signal: PartialFunction[Int, Int] = {  
    case 0 =>  0  
    case x if x > 0 => x - 1   
    case x if x < 00 => x + 1  
}  
```

```scala
def receive={
  case s: Shape=>
  ..
  case Exit =>
  ...
  case unexpected =>    //默认的值
  ...
}
```

语法糖

```scala
 var a=new Json("s")
 a it "s" //与a.it("s")一样
 class Json(s:String){
    def it(s:String): Unit ={
      print(s)
    }
  }
```

只有在方法参数只有一个的时候

函数

偏应用函数

Scala 偏应用函数是一种表达式，你不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。

```scala
import java.util.Date

object Test {
   def main(args: Array[String]) {
      val date = new Date
      val logWithDateBound = log(date, _ : String)

      logWithDateBound("message1" )
      Thread.sleep(1000)
      logWithDateBound("message2" )
      Thread.sleep(1000)
      logWithDateBound("message3" )
   }

   def log(date: Date, message: String)  = {
     println(date + "----" + message)
   }
}
```

绑定第一个 date 参数，第二个参数使用下划线(_)替换缺失的参数列表，并把这个新的函数值的索引的赋给变量。

柯里化函数

将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。

```scala
object Test {
   def main(args: Array[String]) {
      val str1:String = "Hello, "
      val str2:String = "Scala!"
      println( "str1 + str2 = " +  strcat(str1)(str2) )
   }

   def strcat(s1: String)(s2: String) = {
      s1 + s2
   }
}
```

右边括号参数只有一个时，可以使用{}代替()。

函数字面量

```scala
val f1:(Int,String) => String =(i,s)=>i+s
val f2:Function2[Int,String,String]=(i,s)=>i+s
```

以上两种表达方式一致



对于只有一个参数的函数，可以省略参数外围的 `()`。

如果参数只在 `=>` 右侧出现一次，则可以用 `_` 替代它。

注意这些简写方法只有在参数类型已知的情况下才有效。

### import

Scala 中的 import 语句的使用比较灵感，可以用在代码的任意部分

- `import Java.lang._`引入包下所有类
- `import bobsdelights.Fruits._`静态引用，使用Fruits内部的类
- `import Fruits.{Apple,Orange}`部分引用，只引用Apple,Orange
- `import Fruits.{Apple=>MaIntosh,Orange}`重命名，Apple重命名为MaIntosh，包名也可以，将重命名为_，则起到了隐藏效果

### 泛型

https://fangjian0423.github.io/2015/06/07/scala-generic/

[_]等于Java中的<?>

### copy方法

Scala 编译器为 case class 添加了一个 Copy 方法，这个 copy 方法可以用来构造类对象的一个可以修改的拷贝。这对于使用已有的对象构造一个新实例非常方便，你只要修新创建实例的某些参数即可。 比如我们想创建一个和 op 类似的新的实例，只想修改+为-,可以使用：

```scala
  scala> op.copy(operator="-")
    res4: BinOp = BinOp(-,Number(1.0),Var(x))
```

以上这些惯例带来了很多便利，这些便利也需要一些小小的代价，就是需要在 class 前面使用 case 关键字，而构造后的类由于自动添加了一些方法而变大了些。case class 带来的最大的好处是它们支持模式识别。