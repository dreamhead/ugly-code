代码之丑（九）
===

这是一个让我纠结了很久的话题：缩进。
```c++
  for (int j = 0; j < attributes.size(); j++) {
    Attr *attr = attributes.get(j);
    if (attr == NULL ) {
      continue;
    }

    int IsCallFunc = -1;
    if(attr->status() == STATUS_NEW || attr->status() == STATUS_MODIFIED) {
      if(strcmp(attr->attrID(), "CallFunc") == 0) {
        if(0 == strcmp(attr->attrValue(), "1")) {
          IsCallFunc = 1;
        } else if(0 == strcmp(attr->attrValue(), "0")) {
          IsCallFunc = 0;
        }
      }
    } else if (attr->status() == STATUS_DELETED) {
      IsCallFunc = 0;
    }

    ...
  }
```

不是因为它不够“丑”，而是表现它不那么容易。找出一段能表现它特点的代码轻而易举，但放到一篇文章里，大片的代码还是容易让人怀疑我在偷懒。

咬咬牙，我还是拿出了一段。就是这样一段已经缩进很多层的代码，实际上，也只不过是一个更大缩进中的一小段。而且，省略号告诉我们，后面还有。

回到这段代码上，能出现多层缩进，for循环功不可没。出现这种循环，很多情况下，都是对一个集合进行处理，而循环里的内容，就是对集合里的每一个元素进行处理。这里也不例外。所以，我们先做一次提取：
```c++
  for (int j = 0; j < attributes.size(); j++) {
    processAttr(attributes.get(j));
  }

  void processAttr(Attr *attr) {
    if (attr == NULL ) {
      return;
    }

    int IsCallFunc = -1;
    if(attr->status() == STATUS_NEW || attr->status() == STATUS_MODIFIED) {
      if(strcmp(attr->attrID(), "CallFunc") == 0) {
        if(0 == strcmp(attr->attrValue(), "1")) {
          IsCallFunc = 1;
        } else if(0 == strcmp(attr->attrValue(), "0")) {
          IsCallFunc = 0;
        }
      }
    } else if (attr->status() == STATUS_DELETED) {
      IsCallFunc = 0;
    }

    ...
  }
```

至此，我们去掉了一层缩进，而且因为这个提取，语义也变得很清晰：这个新函数只是处理集合里的一个元素。

接下来，这个函数里面长长的代码是对IsCallFunc进行设值，后面省略的部分会根据这里求出的结果进行处理。所以，这里把processAttr进一步分拆：
```c++
void processAttr(Attr *attr) {
  if (attr == NULL ) {
    return;
  }

  int IsCallFunc = isCallFunc(attr);
  …
}

int isCallFunc(Attr *attr) {
  if(attr->status() == STATUS_NEW 
  || attr->status() == STATUS_MODIFIED) {
    if(strcmp(attr->attrID(), "CallFunc") == 0) {
      if(0 == strcmp(attr->attrValue(), "1")) {
    return 1;
      } else if(0 == strcmp(attr->attrValue(), "0")) {
    return 0;
      }
    }
  } else if (attr->status() == STATUS_DELETED) {
    return 0;
  }

  return -1;
}
```

isCallFunc的代码已经独立出来，但依然有多层缩进，分解可以继续：
```c++
  int isCallFunc(Attr *attr) {
    if(attr->status() == STATUS_NEW || attr->status() == STATUS_MODIFIED) {
      return isCallFuncForNewOrModified(attr);
    } else if (attr->status() == STATUS_DELETED) {
      return 0;
    }

    return -1;
  }

  int isCallFuncForNewOrModified(Attr *attr) {
    if(strcmp(attr->attrID(), "CallFunc") == 0) {
      if(0 == strcmp(attr->attrValue(), "1")) {
        return 1;
      } else if(0 == strcmp(attr->attrValue(), "0")) {
        return 0;
      }
    }

    return -1;
  }
```

缩进还有，如果有兴趣，还可以继续分解。这里就到此为止吧！

多层缩进是那种放在代码海一眼就可以认出来的代码，用一条简单的规则就可以限制它：
* 不允许出现多层缩进。

按照我的喜好，3就意味着“多”了。对于switch，我会给予特别的关照，因为switch一旦出场，条件少了，你都不好意思和人打招呼，再缩进就找不到北了。于是，对switch而言，我以为2就是多了，也就是说，switch里面就别再缩进了。

写代码，千万别退让太多。