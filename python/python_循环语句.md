## 循环语句

## for循环

list或tuple可以表示一个有序集合。如果我们想依次访问一个list中的每一个元素呢？比如 list：

```python
L = ['Adam', 'Lisa', 'Bart']
print L[0]
print L[1]
print L[2]
```

如果list只包含几个元素，这样写还行，如果list包含1万个元素，我们就不可能写1万行print。

这时，循环就派上用场了。

Python的 for 循环就可以依次把list或tuple的每个元素迭代出来：

```python
L = ['Adam', 'Lisa', 'Bart']
for name in L:
    print name
```

**注意:**  name 这个变量是在 for 循环中定义的，意思是，依次取出list中的每一个元素，并把元素赋值给 name，然后执行for循环体（就是缩进的代码块）。

这样一来，遍历一个list或tuple就非常容易了。

## while循环

和 for 循环不同的另一种循环是 while 循环，while 循环不会迭代 list 或 tuple 的元素，而是根据表达式判断循环是否结束。

比如要从 0 开始打印不大于 N 的整数：

```python
N = 10
x = 0
while x < N:
    print x
    x = x + 1
```

while循环每次先判断 x < N，如果为True，则执行循环体的代码块，否则，退出循环。

在循环体内，x = x + 1 会让 x 不断增加，最终因为 x < N 不成立而退出循环。

如果没有这一个语句，**while循环在判断 x < N 时总是为True**，就会无限循环下去，变成死循环，所以要特别留意while循环的退出条件。

## break

用 for 循环或者 while 循环时，如果要在循环体内直接退出循环，可以使用 break 语句。

比如计算1至100的整数和，我们用while来实现：

```python
sum = 0
x = 1
while True:
    sum = sum + x
    x = x + 1
    if x > 100:
        break
print sum
```

咋一看， while True 就是一个死循环，但是在循环体内，我们还判断了 x > 100 条件成立时，用break语句退出循环，这样也可以实现循环的结束。

## continue

在循环过程中，可以用break退出当前循环，还可以用continue跳过后续循环代码，继续下一次循环。

假设我们已经写好了利用for循环计算平均分的代码：

```python
L = [75, 98, 59, 81, 66, 43, 69, 85]
sum = 0.0
n = 0
for x in L:
    sum = sum + x
    n = n + 1
print sum / n
```

现在老师只想统计及格分数的平均分，就要把 x < 60 的分数剔除掉，这时，利用 continue，可以做到当 x < 60的时候，不继续执行循环体的后续代码，直接进入下一次循环：

```python
for x in L:
    if x < 60:
        continue
    sum = sum + x
    n = n + 1
```

## 多重循环

在循环内部，还可以嵌套循环，我们来看一个例子：

```
for x in ['A', 'B', 'C']:
    for y in ['1', '2', '3']:
        print x + y
```

x 每循环一次，y 就会循环 3 次，这样，我们可以打印出一个全排列：

```
A1
A2
A3
B1
B2
B3
C1
C2
C3
```

