代码之丑（二）
===

这是一个长长的判断条件：
```c++
  if ( strcmp(rec.type, "PreDropGroupSubs") == 0
    || strcmp(rec.type, "StopUserGroupSubsCancel") == 0
    || strcmp(rec.type, "QFStopUserGroupSubs") == 0
    || strcmp(rec.type, "QFStopUserGroupSubsCancel") == 0
    || strcmp(rec.type, "QZStopUserGroupSubs") == 0
    || strcmp(rec.type, "QZStopUserGroupSubsCancel") == 0
    || strcmp(rec.type, "SQStopUserGroupSubs") == 0
    || strcmp(rec.type, "SQStopUserGroupSubsCancel") == 0
    || strcmp(rec.type, "StopUseGroupSubs") == 0
    || strcmp(rec.type, "PreDropGroupSubsCancel") == 0)
```

之所以注意到它，因为最后两个条件是最新修改里面加入的，换句话说，这不是一次写就的代码。单就这一次而言，只改了两行，这是可以接受的。但这是遗留代码。每次可能只改了一两行，通常我们会不只一次踏入这片土地。经年累月，代码成了这个样子。

这并非我接触过的最长的判断条件，这种代码极大的开拓了我的视野。现在的我，即便面对的是一屏无法容纳的条件，也可以坦然面对了，虽然显示器越来越大。

其实，如果这个判断条件是这个函数里仅有的东西，我也就忍了。遗憾的是，大多数情况下，这只不过是一个更大函数中的一小段而已。

为了让这段代码可以接受一些，我们不妨稍做封装：
```c++
  bool shouldExecute(Record& rec) {
    return (strcmp(rec.type, "PreDropGroupSubs") == 0
      || strcmp(rec.type, "StopUserGroupSubsCancel") == 0
      || strcmp(rec.type, "QFStopUserGroupSubs") == 0
      || strcmp(rec.type, "QFStopUserGroupSubsCancel") == 0
      || strcmp(rec.type, "QZStopUserGroupSubs") == 0
      || strcmp(rec.type, "QZStopUserGroupSubsCancel") == 0
      || strcmp(rec.type, "SQStopUserGroupSubs") == 0
      || strcmp(rec.type, "SQStopUserGroupSubsCancel") == 0
      || strcmp(rec.type, "StopUseGroupSubs") == 0
      || strcmp(rec.type, "PreDropGroupSubsCancel") == 0);
  }

  if (shouldExecute(rec)) {
    ...
  }
```

现在，虽然条件依然还是很多，但和原来庞大的函数相比，至少它已经被控制在一个相对较小的函数里了。更重要的是，通过函数名，我们终于有机会说出这段代码判断的是什么了。

提取函数把这段代码混乱的条件分离开来，它还是可以继续改进的。比如，我们把判断的条件进一步提取：
```c++
bool shouldExecute(Record& rec) {
  static const char* execute_type[] = {
    "PreDropGroupSubs",
    "StopUserGroupSubsCancel",
    "QFStopUserGroupSubs",
    "QFStopUserGroupSubsCancel",
    "QZStopUserGroupSubs",
    "QZStopUserGroupSubsCancel",
    "SQStopUserGroupSubs",
    "SQStopUserGroupSubsCancel",
    "StopUseGroupSubs",
    "PreDropGroupSubsCancel"
  };

  int size = ARRAY_SIZE(execute_type);
  for (int i = 0; i < size; i++) {
    if (strcmp(rec.type, execute_type[i]) == 0) {
      return true;
    }
  }

  return false;
}
```

这样的话，再加一个新的type，只要在数组中增加一个新的元素即可。如果我们有兴趣的话，还可以进一步对这段代码进行封装，把这个type列表变成声明式，进一步提高代码的可读性。

发现这种代码很容易，只要看到在长长的判断条件，就是它了。要限制这种代码的存在，我们只要以设定一个简单的规则：

* 判断条件里面不允许多个条件的组合

在实际的应用中，我们会把“3”定义为“多”，也就是如果有两个条件的组合，可以接受，如果是三个，还是改吧！

虽然通过不断调整，这段代码已经不同于之前，但它依然不是我们心目中的理想代码。出现这种代码，往往意味背后有更严重的设计问题。不过，它并不是这里讨论的内容，这里的讨论就到此为止吧！