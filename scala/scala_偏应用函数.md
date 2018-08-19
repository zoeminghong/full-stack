# 偏应用函数

Scala 偏应用函数是一种表达式，你不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。

如下实例，我们打印日志信息：

```scala
import java.util.Date

object Test {
   def main(args: Array[String]) {
      val date = new Date
      log(date, "message1" )
      Thread.sleep(1000)
      log(date, "message2" )
      Thread.sleep(1000)
      log(date, "message3" )
   }

   def log(date: Date, message: String)  = {
     println(date + "----" + message)
   }
}
```

执行以上代码，输出结果为：

```shell
$ scalac Test.scala
$ scala Test
Mon Dec 02 12:52:41 CST 2013----message1
Mon Dec 02 12:52:41 CST 2013----message2
Mon Dec 02 12:52:41 CST 2013----message3
```

实例中，log() 方法接收两个参数：date 和 message。我们在程序执行时调用了三次，参数 date 值都相同，message 不同。

我们可以使用偏应用函数优化以上方法，绑定第一个 date 参数，第二个参数使用下划线(_)替换缺失的参数列表，并把这个新的函数值的索引的赋给变量。以上实例修改如下：

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

执行以上代码，输出结果为：

```shell
$ scalac Test.scala
$ scala Test
Mon Dec 02 12:53:56 CST 2013----message1
Mon Dec 02 12:53:56 CST 2013----message2
Mon Dec 02 12:53:56 CST 2013----message3
```

