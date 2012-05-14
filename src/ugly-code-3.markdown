代码之丑（三）
===

又见switch：
```c
  switch(firstChar) {
  case ‘N’:
    nextFirstChar = ‘O’;
    break;
  case ‘O’:
    nextFirstChar = ‘P’;
    break;
  case ‘P’:
    nextFirstChar = ‘Q’;
    break;
  case ‘Q’:
    nextFirstChar = ‘R’;
    break;
  case ‘R’:
    nextFirstChar = ‘S’;
    break;
  case ‘S’:
    nextFirstChar = ‘T’;
    break;
  case ‘T’:
    throw Ebusiness();
  default:
  }
```

出于多年编程养成的条件反射，我对于switch总会给予更多的关照。研习面向对象编程之后，看见switch就会想到多态，遗憾的是，这段代码和多态没什么关系。仔细阅读这段代码，我找出了其中的规律，nextFirstChar就是firstChar的下一个字符。于是，我改写了这段代码：
```c
  switch(firstChar) {
  case ‘N’:
  case ‘O’:
  case ‘P’:
  case ‘Q’:
  case ‘R’:
    nextFirstChar = firstChar + 1;
    break;
  case ‘T’:
    throw Ebusiness();
  default:
  }
```

现在，至少看起来，这段代码已经比原来短了不少。当然这么做基于一个前提，也就是这些字母编码的顺序确确实实连续的。从理论上说，开始那段代码适用性更强。但在实际开发中，我们碰到字母不连续编码的概率趋近于0。

但这段代码究竟是如何产生的呢？我开始研读上下文，原来这段代码是用当前ID产生下一个ID的，比如当前是N0000，下一个就是N0001。如果数字满了，就改变字母，比如当前ID是R9999，下一个就是T0000。在这里，字母也就相当于一位数字，根据情况进行进位，所以有了这段代码。

代码上的注释告诉我，字母的序列只有从N到T，根据这个提示，我再次改写了这段代码：
```c
  if (firstChar >= ‘N’ && firstChar <= ‘S') {
    nextFirstChar = firstChar + 1;
  } else {
    throw Ebusiness();
  }
```

这里统一处理了字母为T和default的情形，严格说来，这和原有代码并不完全等价。但这是了解了需求后做出的决定，换句话说，原有代码在这里的处理中存在漏洞。

修改这段代码，只是运用了非常简单的编程技巧。遗憾的是，即便如此简单的编程技巧，也不是所有开发人员都驾轻就熟的，很多人更习惯于“平铺直叙”。这种直白造就了代码中的许多鸿篇巨制。我听过不少“编程是体力活”的抱怨，不过，能把写程序干成体力活，也着实不值得同情。写程序，不动脑子，不体力才怪。

无论何时何地，只要switch出现在眼前，请提高警惕，那里多半有坑。