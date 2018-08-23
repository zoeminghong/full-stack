## if语句

计算机之所以能做很多自动化的任务，因为它可以自己做条件判断。

比如，输入用户年龄，根据年龄打印不同的内容，在Python程序中，可以用if语句实现：

```python
age = 20
if age >= 18:
    print 'your age is', age
    print 'adult'
print 'END'
```

**注意:** Python代码的缩进规则。具有相同缩进的代码被视为代码块，上面的3，4行 print 语句就构成一个代码块（但不包括第5行的print）。如果 if 语句判断为 True，就会执行这个代码块。

缩进请严格按照Python的习惯写法：4个空格，不要使用Tab，更不要混合Tab和空格，否则很容易造成因为缩进引起的语法错误。

**注意**: if 语句后接表达式，然后用`:`表示代码块开始。

如果你在Python交互环境下敲代码，还要特别留意缩进，并且退出缩进需要多敲一行回车：

```python
>>> age = 20
>>> if age >= 18:
...     print 'your age is', age
...     print 'adult'
...
your age is 20
adult
```

当 if 语句判断表达式的结果为 True 时，就会执行 if 包含的代码块：

```
if age >= 18:
    print 'adult'
```

如果我们想判断年龄在18岁以下时，打印出 'teenager'，怎么办？

方法是再写一个 if:

```
if age < 18:
    print 'teenager'
```

或者用 not 运算：

```
if not age >= 18:
    print 'teenager'
```

细心的同学可以发现，这两种条件判断是“非此即彼”的，要么符合条件1，要么符合条件2，因此，完全可以用一个 if ... else ... 语句把它们统一起来：

```
if age >= 18:
    print 'adult'
else:
    print 'teenager'
```

利用 if ... else ... 语句，我们可以根据条件表达式的值为 True 或者 False ，分别执行 if 代码块或者 else 代码块。

**注意:** else 后面有个“:”。

## if-elif-else

有的时候，一个 if ... else ... 还不够用。比如，根据年龄的划分：

```
条件1：18岁或以上：adult
条件2：6岁或以上：teenager
条件3：6岁以下：kid
```

我们可以用一个 if age >= 18 判断是否符合条件1，如果不符合，再通过一个 if 判断 age >= 6 来判断是否符合条件2，否则，执行条件3：

```python
if age >= 18:
    print 'adult'
else:
    if age >= 6:
        print 'teenager'
    else:
        print 'kid'
```

这样写出来，我们就得到了一个两层嵌套的 if ... else ... 语句。这个逻辑没有问题，但是，如果继续增加条件，比如3岁以下是 baby：

```python
if age >= 18:
    print 'adult'
else:
    if age >= 6:
        print 'teenager'
    else:
        if age >= 3:
            print 'kid'
        else:
            print 'baby'
```

这种缩进只会越来越多，代码也会越来越难看。

要避免嵌套结构的 if ... else ...，我们可以用 if ... 多个elif ... else ...的结构，一次写完所有的规则：

```python
if age >= 18:
    print 'adult'
elif age >= 6:
    print 'teenager'
elif age >= 3:
    print 'kid'
else:
    print 'baby'
```

elif 意思就是 else if。这样一来，我们就写出了结构非常清晰的一系列条件判断。

**特别注意:** 这一系列条件判断会从上到下依次判断，如果某个判断为 True，执行完对应的代码块，后面的条件判断就直接忽略，不再执行了。