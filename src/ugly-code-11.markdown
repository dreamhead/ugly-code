代码之丑（十一）
===

全局变量永远是不受欢迎的，因为它会带来太多的问题，所以，诸如Java这样的程序设计语言干脆摒弃了全局变量。一旦我们有机会面对全局变量，想都不要想，干掉它。
```c
if (IDLE == g_status) {
  ...
}
```

那个g打头的家伙就是全局变量，它就是我们的靶子。第一直觉，我们不要直接访问全局变量，那就用函数把它封装起来：
```c
int getCurrentStatus() {
  return gStatus;
}
```

于是，代码变身了：
```c
if (getCurrentStatus() == IDLE) {
  ...
}
```

把变量封装成函数，从某种角度说，这是一种进步。但我想说，这还不够。这只是一种简单的封装，本质上来说，这与直接暴露数据差别不大，我们需要更好的封装，通常的做法是封装出行为。行为从哪来，从实际需求来。

就以上面这段代码为例，我们封装了status，其实，它的目的是为了与IDLE状态相比较，这就是一种行为，我们可以这样封装：
```c
bool isCurrentStatus(int status) {
  return status == g_status;
}

if (isCurrentStatus(IDLE)) {
  ...
}
```

还有一种修改方式，既然IDLE是一个固定的常量，索性把它也隐藏起来：
```c
bool isIdle() {
  return IDLE == g_status;
}

if (isIdle()) {
  ... 
}
```

实际上，这种封装出行为的方式不仅仅适用于全局变量，把数据拿出来再用的情形也是经常可以见到的：
```c++
if (machine.getStatus() == IDLE) {
  ...
}
```

封装的方式同上面一样，这里选择一种实现：
```c++
class Machine {
  ...
  bool isIdle() {
    return status == IDLE;
  }
}

if (machine.isIdle()) {
  ...
}
```

封装，就得封装出个行为来。