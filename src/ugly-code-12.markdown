代码之丑（十二）
===

诸位Java程序员，想必大家对SimpleDateFormat并不陌生。不过，你是否知道，SimpleDateFormat不是线程安全的（thread safe）。这意味着，下面的代码是错误的：
```java
class Sample {
  private static final DateFormat format = new SimpleDateFormat("yyyy.MM.dd");

  public String getCurrentDateText() {
    return format.format(new Date());
  }
}
```
从功能的角度上看，单独执行这段代码是没有问题的，但放到多线程环境下，因为SimpleDateFormat不是线程安全的，这段代码就会出错。所以，要想让这段代码正确，我们只要稍做微调：
```java
public class Sample {
    public String getCurrentDateText() {
        return new SimpleDateFormat("yyyy.MM.dd").format(new Date());
    }
}
```
不知你是否注意到，这里的调整只是由原来的共享format这个变量，变成了每次调用这个方法时创建出一个新的SimpleDateFormat变量。

作为一个专业程序员，我们当然知道，相比于共享一个变量的开销要比每次创建小。之所以我们必须这么做，是因为SimpleDateFormat不是线程安全的。但从SimpleDateFormat提供给我们的接口上来看，实在让人看不出它与线程安全有和相干。那接下来，我们就要打开JDK的源码，看一下其中的代码之丑。

如果你手头没有JDK的源码，这里是个不错的参考。

在format方法里，有这样一段代码：
```java
  calendar.setTime(date);
```
其中，calendar是DateFormat的protected字段。这条语句改变了calendar，稍后，calendar还会用到（在subFormat方法里），而这就是引发问题的根源。

想象一下，在一个多线程环境下，有两个线程持有了同一个SimpleDateFormat的实例，分别调用format方法：

* 线程1调用format方法，改变了calendar这个字段。
* 中断来了。
* 线程2开始执行，它也改变了calendar。
* 又中断了。
* 线程1回来了，此时，calendar已然不是它所设的值，而是走上了线程2设计的道路。
* **BANG！！！**

稍微花点时间分析一下format的实现，我们便不难发现，用到calendar，唯一的好处，就是在调用subFormat时，少了一个参数，却带来了这许多的问题。其实，**只要在这里用一个局部变量**，一路传递下去，所有问题都将迎刃而解。

这个问题背后隐藏着一个更为重要的问题：无状态。

无状态方法的好处之一，就是它在各种环境下，都可以安全的调用。衡量一个方法是否是有状态的，就看它是否改动了其它的东西，比如全局变量，比如实例的字段。format方法在运行过程中改动了SimpleDateFormat的calendar字段，所以，它是有状态的。

写程序，我们要尽量编写无状态方法。