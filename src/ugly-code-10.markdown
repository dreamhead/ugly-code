代码之丑（十）
===

C语言出现之初，跨平台是个极大的卖点。于是，我们有机会看到这样的代码：
```c
  int sys_old_mmap(struct tcb *tcp)
  {
    long u_arg[6];

  #if defined(IA64)
    int i, v;
    for (i = 0; i < 6; i++)
      if (umove(tcp, tcp->u_arg[0] + (i * sizeof(int)), &v) == -1)
        return 0;
      else
        u_arg[i] = v;
  #elif defined(SH) || defined(SH64)
    int i;
    for (i=0; i<6; i++)
      u_arg[i] = tcp->u_arg[i];
  #else
    if (umoven(tcp, tcp->u_arg[0], sizeof(u_arg), (char *) u_arg) == -1)
      return 0;
  #endif  // defined(IA64)
    return print_mmap(tcp, u_arg);
  }
```

你已经知道了我要说什么了，是的，条件编译。

条件编译在解决跨平台的问题上，确实是个利器，但这么用条件编译，就把它变成了一柄双刃剑。Robert Martin在《Clean Code》里告诉我们，函数应该只做一件事。从逻辑上来说，这段代码是做了一件事。但是，它还有另外一个维度，也就是条件编译的条件。这个维度的存在让一件事变成了多件事。

无他，提取函数吧！
```c
  int sys_old_mmap(struct tcb *tcp)
  {
  #if defined(IA64)
    return sys_old_mmap_for_ia64(tcp);
  #elif defined(SH) || defined(SH64)
    return sys_old_mmap_for_sh_or_sh64(tcp);
  #else
    return default_sys_old_mmap(tcp);
  #endif  // defined(IA64)
  }

  int sys_old_mmap_for_ia64(struct tcb *tcp) {
    long u_arg[6];
    int i, v;
    for (i = 0; i < 6; i++)
      if (umove(tcp, tcp->u_arg[0] + (i * sizeof(int)), &v) == -1)
        return 0;
      else
        u_arg[i] = v;

    return print_mmap(tcp, u_arg);
  }

  int sys_old_mmap_for_sh_or_sh64(struct tcb *tcp) {
    long u_arg[6];
    int i;
    for (i=0; i<6; i++)
      u_arg[i] = tcp->u_arg[i];

    return print_mmap(tcp, u_arg);
  }

  int default_sys_old_mmap(struct tcb *tcp) {
    long u_arg[6];
    if (umoven(tcp, tcp->u_arg[0], sizeof(u_arg), (char *) u_arg) == -1)
      return 0;

    return print_mmap(tcp, u_arg);
  }
```

好，经过一番分解，函数比原来的规模小了许多，更重要的是，我们把针对不同的条件做法已经拆解开来了。

相对于原来的版本，这是一段可以接受的代码。制约原有代码的出现，我们也可以用一个简单的规则：

条件编译里面不允许包含多条执行语句。
不过，这还不是终点。针对上面的情况，这种改法没有问题，因为提取出来的小函数在各个平台上都可以编译，但如果涉及到特定平台的操作，简单的提取就不起作用了。比如，用到了Windows API的代码在Linux上恐怕连编译这关都过不了。

一种可能的解决方案是，把不同条件的内容放进不同的文件。
```c
  int sys_old_mmap(struct tcb *tcp) {
    long u_arg[6];
    int i, v;
    for (i = 0; i < 6; i++)
      if (umove(tcp, tcp->u_arg[0] + (i * sizeof(int)), &v) == -1)
        return 0;
      else
        u_arg[i] = v;

    return print_mmap(tcp, u_arg);
  }
```
  (ia64.c)


```c
  int sys_old_mmap(struct tcb *tcp) {
    long u_arg[6];
    int i;
    for (i=0; i<6; i++)
      u_arg[i] = tcp->u_arg[i];

    return print_mmap(tcp, u_arg);
  }
```
  (sh.c)


```c
  int sys_old_mmap(struct tcb *tcp) {
    long u_arg[6];
    if (umoven(tcp, tcp->u_arg[0], sizeof(u_arg), (char *) u_arg) == -1)
      return 0;

    return print_mmap(tcp, u_arg);
  }
```
  (default.c)


同之前的一个差别在于，不再有一个统一的sys_old_mmap，而是大家有了各自的sys_old_mmap，分别放在了对应的文件里。构建的时候，我们可以根据不同的条件编译链接不同的文件，这样就可以回避前面提到的问题。

之前谈及的那些丑陋代码大多是在一个文件内部做着各种各样的变换，而这次的变动显然大了很多，甚至需要配合构建过程的修改。为了对付丑陋的代码，我们总是有办法的。

即便劳神费力的修改构建过程，我依然认为是值得的。其实，我们想要的，无非是明天的日子好过一些。