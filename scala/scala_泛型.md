# 泛型

把带有一个或者多个类型参数的类，叫作泛型类。如果你把类型参数替换成实际的类型，将得到一个普通的类。比如Student[String]

## 泛型类

```scala
class Student[T](val name: T) {
      println(name)
}
```

## 泛型函数

```scala
def getStudentInfo[T](stu: Array[T]) = stu(stu.length / 2)
```

## 类型变量界定—上限

类似于Java中如下的操作

```scala
<T extends Test>
//或用通配符的形式：
<? extends Test>
```

对应于Scala中的实现是

```scala
[T <: Test] 
//或用通配符: 
[_ <: Test]
```

`e.g.`

```scala
class Student[T <: Comparable[T]](val first: T, val second: T){
    def smaller = if (first.compareTo(second) < 0) first else second
}

val studentString = new Student[String]("limu","john")
println(studentString.smaller)

val studentInt = new Student[Integer](1,2)
println(studentInt.smaller)
```

## 类型变量界定—下限

与上限类似，只不过是`:>`

## 类型通配符

 类型通配符是指在使用时不具体指定它属于某个类，而是只知道其大致的类型范围，通过`_ <:` 达到类型通配的目的。

```scala
def typeWildcard: Unit ={
    class Person(val name:String){
        override def toString()=name
    }
    class Student(name:String) extends Person(name)
    class Teacher(name:String) extends Person(name)
    class Pair[T](val first:T,val second:T){
        override def toString()="first:"+first+", second: "+second;
    }
    //Pair的类型参数限定为[_<:Person]，即输入的类为Person及其子类
    //类型通配符和一般的泛型定义不一样，泛型在类定义时使用，而类型通配符号在使用类时使用
    def makeFriends(p:Pair[_<:Person])={
        println(p.first +" is making friend with "+ p.second)
    }
    makeFriends(new Pair(new Student("john"),new Teacher("摇摆少年梦")))
}
```

> 注意类型通配符与类型变量界定的区别之处，在于一个是类型内部泛型，一个变量是泛型

## 视图界定

使用类型变量界定，该类型必须是上限的子类，但存在一种情况某个类型其不是该上限的子类，但通过隐式转换可以得到该上限的子类，这种实现就叫视图界定，视图界定利用`<%`符号来实现。

先看下面的一个例子：

```scala
class Student[T <: Comparable[T]](val first: T, val second: T) {
    def smaller = if (first.compareTo(second) < 0) first else second
}
val student = new Student[Int](4,2)
println(student.smaller)
```

可惜，如果我们尝试用Student(4,2)五实现，编译器会报错。因为Int和Integer不一样，Integer是包装类型，但是Scala的Int并没有实现Comparable。
不过RichInt实现了Comparable[Int]，同时还有一个Int到RichInt的隐式转换。解决途径就是视图界定。

```scala
class Student[T <% Comparable[T]](val first: T, val second: T) {
    def smaller = if (first.compareTo(second) < 0) first else second
}
```

使用视图界定满足条件：

- 存在隐式转换
- 隐式转换得到的类型时类型界定上限的子类

## 协变和逆变

在Java不支持泛型类型实例与类型不对称的存在，但在Scala中是支持顺势和逆势的转变

```scala
object Demo {
    def main(args: Array[String]): Unit = {
        val myList1:MyList[Person] = new MyList[Person]()
        val myList2:MyList[Person] = new MyList[Student]()

        val yourList1:YourList[Person] = new YourList[Person]()
        val yourList2:YourList[Student] = new YourList[Person]()
    }

    class Person{}
    class Student extends Person{}
    class Teacher extends Person{}
    /**
      * 支持协变的泛型类
      */
    class MyList[+T] {
    }

    /**
      * 支持逆变的泛型类
      */
    class YourList[-T] {
    }
}
```

## Scala与Java

[_]等于Java中的<?>

http://blog.51cto.com/xpleaf/2106971

https://fangjian0423.github.io/2015/06/07/scala-generic/