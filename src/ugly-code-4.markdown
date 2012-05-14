代码之丑（四）
===

这是一个找茬的游戏，下面三段代码的差别在哪：
```c++
  if ( 1 == SignRunToInsert)    {
    RetList->Insert(i, NewCatalog);
  } else {
    RetList->Add(NewCatalog);
  }
```
```c++
  if ( 1 == SignRunToInsert)    {
    RetList->Insert(m, NewCatalog);
  } else {
    RetList->Add(NewCatalog);
  }
```
```c++
  if ( 1 == SignRunToInsert)    {
    RetList->Insert(j, NewPrivNode);
  } else {
    RetList->Add(NewPrivNode);
  }
```

答案时间：除了用到变量之外，完全相同。我想说的是，这是我从一个文件的一次diff中看到的。

不妨设想一下修改这些代码时的情形：费尽九牛二虎之力，我终于找到该在哪改动代码，改了。作为一个有职业操守的程序员，我知道别的地方也需要类似的修改。于是，趁人不备，把刚做修改拷贝了一份，放到另外需要修改的地方。修改了几个变量，编译通过了。世界应该就此清净，至少问题解决了。

好吧！虽然这个程序员有职业操守的程序员，却缺少了些职业技能，至少在挥舞“拷贝粘贴”时，他没有嗅到散发出的臭味。

只要意识到坏味道，修改是件很容易的事，提出一个新函数即可：
```c++
  void AddNode(List& RetList, int SignRunToInsert, int Pos, Node& Node) {
    if ( 1 == SignRunToInsert)    {
      RetList->Insert(Pos, Node);
    } else {
      RetList->Add(Node);
    }
  }
```

于是，原来那三段代码变成了三个调用：
```c++
  AddNode(RetList, SignRunToInsert, i, NewCatalog);
  AddNode(RetList, SignRunToInsert, m, NewCatalog);
  AddNode(RetList, SignRunToInsert, j, NewPrivNode);
```

当然，这种修改只是一个局部的微调，如果有更多的上下文信息，我们可以做得更好。

重复，是最为常见的坏味道。上面这种重复实际上是非常容易发现的，也是很容易修改。但所有这一切的前提是，发现坏味道。

长时间生活在这种代码里面，我们会对坏味道失去嗅觉。更可怕的是，一个初来乍到的嗅觉尚灵敏的人意识到这个问题，那些失去嗅觉的人却告诫他，别乱动，这挺好。

趁嗅觉尚在，请坚持代码正义。