代码之丑（二）（续二）
===

一个叫夏勇杰的朋友看了《代码之丑》（二）》和《续》之后，给我写了封邮件，就原来的问题，给出了自己的解决方案，这里分享一下。

他的思路是把所有判断条件转换成数字，然后，利用常见的位操作的技巧来处理。上代码：
```c++
  enum Type {
    PreDropGroupSubs                 = 1,
    StopUserGroupSubsCancel          = 1 << 1,
    QFStopUserGroupSubs              = 1 << 2,
    QFStopUserGroupSubsCancel        = 1 << 3,
    QZStopUserGroupSubs              = 1 << 4,
    QZStopUserGroupSubsCancel        = 1 << 5,
    SQStopUserGroupSubs              = 1 << 6,
    SQStopUserGroupSubsCancel        = 1 << 7,
    StopUseGroupSubs                 = 1 << 8,
    PreDropGroupSubsCancel           = 1 << 9,  
    // ...
  };

  int canExecute = PreDropGroupSubs | StopUserGroupSubsCancel
                        | QFStopUserGroupSubs | QFStopUserGroupSubsCancel
                        | QZStopUserGroupSubs | QZStopUserGroupSubsCancel
                        | SQStopUserGroupSubs | SQStopUserGroupSubsCancel
                        | StopUseGroupSubs | PreDropGroupSubsCancel;

  bool shouldExecute(Record& record) {
    return ((rec.type & canExecute) != 0);
  }
```

当然，这么做的一个前提是，需要把type的类型转换成数字。这不是太大的问题，只要在整个处理开始的部分做一次转换就可以了。从执行效率上来说，这段代码会比原来的方案高得多：一方面在于字符串比较，另一方面在于原来是循环判断。

写《代码之丑》（二）》的时候，我没想着写《续》，因为那些内容不是在讨论“丑”，而纯粹是编程技巧了。写《续》的时候，我更没想到会有《续二》。这两个续篇都是看过前面内容的朋友驱动出来的，而这就是分享的乐趣，不是吗？