代码之丑（二）（续）
===

sinojelly在《代码之丑（二）》的评论里问了个问题，“把这个type列表变成声明式”，什么样的声明式？

好吧！我承认，我偷懒了，为了省事，一笔带过了。简单理解声明式的风格，就是把描述做什么，而不是怎么做。一个声明式编程的例子是Rails里面的数据关联，为人熟知的has_many和belongs_to。通过声明，模型类就会具备一些数据关联的能力。

具体到实际开发里，声明式编程需要有两个部分：一方面是一些基础的框架性代码，另一方面是应用层面如何使用。框架代码通常来说，都不像应用层面代码那么好理解，但有了这个基础，应用代码就会变得简单许多。

针对之前的那段代码，按照声明性编程风格，我改造了代码，下面是框架部分的代码：

```c++
#define BEGIN_STR_PREDICATE(predicate_name) \
bool predicate_name(const char* field) { \
  static const char* predicate_true_fields[] = {
   
#define STR_PREDICATE_ITEM(item) #item ,

#define END_STR_PREDICATE \
  };\
  \
  int size = ARRAY_SIZE(predicate_true_fields);\
  for (int i = 0; i < size; i++) { \
    if (strcmp(field, predicate_true_fields[i]) == 0) {\
        return true;\
    }\
  }\
\
  return false;\
}
```

这里用到了C/C++常见的宏技巧，为的就是让应用层面的代码写起来更像声明。对比一下之前的函数，就会发现，实际上二者几乎是一样的。有了框架，就该应用了：

```c++
BEGIN_STR_PREDICATE(shouldExecute)
  STR_PREDICATE_ITEM(PreDropGroupSubs)
  STR_PREDICATE_ITEM(StopUserGroupSubsCancel)
  STR_PREDICATE_ITEM(QFStopUserGroupSubs)
  STR_PREDICATE_ITEM(QFStopUserGroupSubsCancel)
  STR_PREDICATE_ITEM(QZStopUserGroupSubs)
  STR_PREDICATE_ITEM(QZStopUserGroupSubsCancel)
  STR_PREDICATE_ITEM(SQStopUserGroupSubs)
  STR_PREDICATE_ITEM(SQStopUserGroupSubsCancel)
  STR_PREDICATE_ITEM(StopUseGroupSubs)
  STR_PREDICATE_ITEM(SQStopUserGroupSubsCancel)
  STR_PREDICATE_ITEM(StopUseGroupSubs)
  STR_PREDICATE_ITEM(PreDropGroupSubsCancel)
END_STR_PREDICATE
```

shouldExecute就此重现出来了。不过，这段代码已经不再像一个函数，而更像一段声明，这就是我们的目标。有了这个基础，实现一个新的函数，不过是做一段新的声明而已。

接下来就是如何使用了，与之前略有差异的是，这里为了更好的通用性，把字符串作为参数传了进去，而不是原来的整个类对象。
```c++
  shouldExecute(r.type);
```

虽然应用代码变得简单了，但写出框架的结构是需要一定基础的。它不像应用代码那样来得平铺直叙，但其实也没那么难，只不过很多人从没有考虑把代码写成这样。只要换个角度去思考，多多练习，也就可以驾轻就熟了。