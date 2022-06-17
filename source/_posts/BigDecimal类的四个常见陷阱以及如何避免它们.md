---
title: BigDecimal类的四个常见陷阱以及如何避免它们
date: 2022-06-11 10:14:48
tags: 
  - Java
---

**在Java中进行货币计算时，您可以使用java.math.BigDecimal--但要注意该类的一些独特挑战。**

在使用Java进行商业计算时，尤其是对于货币，最好使用该类来避免与浮点算术相关的问题，如果使用两个基元类型之一：Float或Double(或它们的盒装类型对应类型)，则可能会遇到这些问题。

事实上，该类包含许多方法，可以满足大多数常见业务计算的要求。

然而，我想提请大家注意BigDecimal类的四个常见陷阱。当您使用常规的BigDecimal API和使用一个新的、可扩展的定制类时，可以避免它们。

那么，让我们从常规的 BigDecimal API 开始。

## 陷阱1: 双重构造函数

考虑下面的例子:

```java
BigDecimal x = new BigDecimal(0.1);
System.out.println("x=" + x);
```

下面是控制台的输出:

```
x=0.1000000000000000055511151231257827021181583404541015625
```

如您所见，此构造函数的结果可能有些不可预测。这是因为浮点数在计算机硬件中表示为基数2(二进制)小数。但是，大多数小数不能完全表示为二进制小数。因此，机器中实际存储的二进制浮点数仅接近您输入的十进制浮点数。因此，传递给Double构造函数的值不完全等于0.1。

相反，字符串构造函数是完全可预测的，并且生成的值恰好等于0.1，不出所料。

```java
BigDecimal y = new BigDecimal("0.1");
System.out.println("y=" + y);
```

现在，控制台输出如下：

```
y=0.1
```

因此，您应该优先使用字符串构造函数，而不是Double构造函数。

## 陷阱2: 静态ValueOf方法

如果使用静态`BigDecimal.valueOf(Double)`方法创建`BigDecimal`，请注意其精度有限。

```java
BigDecimal x = BigDecimal.valueOf(1.01234567890123456789);
BigDecimal y = new BigDecimal("1.01234567890123456789");
System.out.println("x=" + x);
System.out.println("y=" + y);
```

上面的代码生成以下控制台输出：

```java
x=1.0123456789012346
y=1.01234567890123456789
```

在这里，该值丢失了四位小数位，因为Double的精度只有15-17位(Float的精度只有6-9位)，而BigDecimal的精度是任意的(仅受内存限制)。

因此，使用字符串构造函数实际上是一个好主意，因为有效地避免了由双重构造函数引起的两个主要问题。

## 陷阱#3：Equals(BigDecimal)方法

```java
BigDecimal x = new BigDecimal("1");
BigDecimal y = new BigDecimal("1.0");
System.out.println(x.equals(y));
```

控制台输出如下：

```
False
```

此输出是由于BigDecimal由任意精度的未缩放整数值和32位整数小数位数组成，这两个值都必须等于要比较的另一个BigDecimal的相应值。在这种情况下

- X的未缩放值为1，比例为0。
- Y的未缩放值为10，比例为1。

因此，X不等于Y。

因此，不应该使用equals()方法比较BigDecimal的两个实例，而应该使用CompareTo()方法，因为它比较了BigDecimal的两个实例表示的数值(x=1；y=1.0)。下面是一个例子：

```java
System.out.println(x.compareTo(y) == 0);
```

现在，控制台输出如下：

```
True
```

## 陷阱#4：round(MathContext)方法

一些开发人员可能会倾向于使用`round(new MathContext(precision, roundingMode))`方法将BigDecimal舍入为(比方说)两位小数。这不是个好主意。

```java
BigDecimal x = new BigDecimal("12345.6789");
x = x.round(new MathContext(2, RoundingMode.HALF_UP));
System.out.println("x=" + x.toPlainString());
System.out.println("scale=" + x.scale());
```

上面的代码生成以下控制台输出，因此x不是12345.68的预期值，刻度也不是2的预期值：

```
x=12000 
scale=-3
```

该方法不会对小数部分进行舍入，但会将未按比例调整的值舍入到给定的有效位数(从左到右计数)，小数点保持不变，在上面的示例中，小数点的小数位数为-3。

那么，这里发生了什么？

未缩放值(123456789)被四舍五入为两位有效数字(12)，这表示精度为2。但是，由于小数点保持不变，因此此BigDecimal表示的实际值为12000.0000。这也可以写为12000，因为小数点右侧的四个零没有意义。

但规模又如何呢？为什么它是-3而不是0，正如您对值12000的预期？

这是因为这个BigDecimal的无标度值是12，因此，它必须乘以1000，这是10的3次方，(12x103)等于12000。

因此，正小数位数表示小数位数(即小数点右侧的位数)，而负数位数表示小数点左侧的无用位数(在本例中是尾随的零，因为它们只是指示数字小数位数的占位符)。

最后，由BigDecimal表示的数字因此是unscaledValue x 10刻度。

还要注意，上面的代码使用了toPlainString()方法，该方法不以科学记数法(1.2E+4)显示结果。

若要获得12345.68的预期结果，请尝试使用setScale(Scale，oundingMode)方法，例如：

```java
BigDecimal x = new BigDecimal("12345.6789");
x = x.setScale(2, RoundingMode.HALF_UP);
System.out.println("x=" + x));
```

现在控制台输出的就是预期的结果。

现在，控制台输出是预期的：

```java
x=12345.68
```

方法的作用是：根据指定的舍入模式将小数部分舍入到小数点后两位。

顺便说一句，您可以使用ROUND(new MathContext(Precision，oundingMode))方法进行传统的舍入。但这需要您知道计算结果小数点左侧的总位数。请考虑以下示例：

```java
BigDecimal a = new BigDecimal("12345.12345");
BigDecimal b = new BigDecimal("23456.23456");
BigDecimal c = a.multiply(b);
System.out.println("c=" + c);
```

现在，控制台输出如下：

```
c=289570111.3153564320
```

要将c舍入到小数点后两位，您必须使用精度为11的MathContext对象，例如：

```java
BigDecimal d = c.round(new MathContext(11, RoundingMode.HALF_UP));
System.out.println("d=" + d);
```

上面的代码生成以下控制台输出：

```
d=289570111.32
```

小数点左侧的总位数可按如下方式计算：

```java
bigDecimal.precision() - bigDecimal.scale() + newScale
```

- `bigDecimal.precision()` 是未四舍五入的总长度。
- `bigDecimal.scale()` 是未四舍五入小数的长度。
- `newScale` 是您指定的新的小数部分长度。

所以，下面代码:

```java
BigDecimal e = c.round(new MathContext(c.precision() - c.scale() + 2, RoundingMode.HALF_UP));
```

控制台输出

```
e=289570111.32
```

然而，如果你将这个表达式

```java
c.round(new MathContext(c.precision() - c.scale() + 2, RoundingMode.HALF_UP));
```

设置为以下表达式

```java
c.setScale(2, RoundingMode.HALF_UP);
```

很明显，您会选择哪一个来确保代码的可读性和简明性。

