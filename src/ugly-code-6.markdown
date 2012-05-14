代码之丑（六）
===

这是一段长长的C++代码，我的问题是：relaPri、relaSec和 scoutBySec这三个变量在哪里用到了？
```c++
  void DealForServiceA(const char *oprCode, const char *subID, const char *oID, XList *callCicsList) {
    XString relaPri(“NULL”);
    XString relaSec(“NULL”);
    XString scoutBySec(“0”);
    XList *tempList = new XList ;
    callCicsList->Add(tempList);
    tempList->Add(new XString(oprCode));
    tempList->Add(new XString(oID));
    XString *psTelNum = new XString;
    tempList->Add(psTelNum);
    GetServnumberBySubsID(subID, *psTelNum);   
    tempList->Add(new XString(relaPri.table { font-size: 10pt;}c_str()));
    tempList->Add(new XString(relaSec.c_str()));
    tempList->Add(new XString(scoutBySec.c_str()));
  }
```

经过认真仔细的查看，或是使用传说的中“查找”功能，我们发现上面提到的那三个变量只在最后用了一下。

不知道你是否注意到，我在最初特意强调了一下这是C++代码。这意味着，变量可以随用随声明，而不必像传统的C程序那样，只能在函数的开头把函数内部用到的变量一口气声明。 那么 ，我们就让声明和使用团聚吧！
```c++
    XString relaPri(“NULL”);
    tempList->Add(new XString(relaPri.c_str()));
    XString relaSec(“NULL”);
    tempList->Add(new XString(relaSec.c_str()));
    XString scoutBySec(“0”);
    tempList->Add(new XString(scoutBySec.c_str()));
```

当声明和使用走到一起，我们的观察就有了新的视角，其实，这几个变量完全是可以不声明的，于是，代码再进一步：
```c++
    tempList->Add(new XString(“NULL”));
    tempList->Add(new XString(“NULL”));
    tempList->Add(new XString(“0”));
```

看到这里，我们就可以看出原来的做法到底有多么浪费：浪费时间给变量起名字——我们都知道，起个好名字不容易，也浪费了时间在执行上，修改前的代码创建了两个XString对象，而修改后，只创建了一个对象。

或许，你会觉得，有个变量会让我们了解这里实际上填加的内容到底是什么。不过，也许一个好的函数命名才是更好的选择，比如 addRelaPri。这个疑问会揭示出这段代码存在另外一个问题，直接使用基本的数据结构而没有进行封装。不过，这不是这里讨论的目标，就到此打住吧！

根据这段代码的调整，我们得出一条规则：

* 代码的声明和使用应尽量接近。

有的C程序员会暗自念叨，这个要求对C程序来说，简直太不合情理了。好吧！我承认，从语言的角度来说，是这样的。但是，我们需要仔细想想，为什么对于C语言来说，变量的声明和使用会距离遥远。通常，遥远的背后意味着硕大的函数，这才是让声明和使用天各一方的重要原因。

在干净代码的世界里，大函数永远是不受欢迎的。为了让声明和使用尽早团聚，请把函数写小。