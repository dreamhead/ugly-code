代码之丑（十三）
===

第一次见到这样的代码时，我的第一感觉是，它真复杂：
```java
  List<Map<String, String>> configurations;
```
可只要理性稍一回归，便不难察觉，它少东西了。少什么了呢？

看看这段代码如何使用，下面是一个缩略的版本：
```java
for (Map<String, String> configuration : configurations) {
  for (Map.Entry<String, String> entry : configuration.entrySet()) {
    System.out.println(entry.getKey() + " " + entry.getValue());
  }
}
```
说白了很简单，其实就是要拿到存在Map里的键值对。在面向对象的程序语言中，有一种神奇的构造，叫做类，而它有一个很重要的特点叫做封装。是的，这段代码少了类，少了封装。闲言少叙，封装起来：
```java
public class ConfigurationItem {
  private String name;
  private String value;
  ...
}
```
于是，那个容器嵌套容器的声明变成了
```java
  List<ConfigurationItem> configurations;
```
有了类，有了封装，我们就可以再进一步进行封装，比如前面那段代码里的
```java
  entry.getKey() + " " + entry.getValue()
```
实际上，可能只是为了得到这一项的字符串表示而已，那就不如直接提供一个方法：
```java
public class ConfigurationItem {
  ...
  @Override
  public String toString() {
    return name + " " + value;
  }
}
```
于是，前面那个双重for循环就变化了：
```java
  for (ConfigurationItem item : configurationItems) {
    System.out.println(item);
  }
```
单以丑陋而言，这段代码还算不上此类的极致，三五层嵌套也是有的，如果某些貌似负责的程序员再给每层取值都加上非空判定，那场面可是相当壮观的。

当容器开始嵌套容器，请考虑封装。