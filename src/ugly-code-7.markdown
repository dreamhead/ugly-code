代码之丑（七）
===

这是一段用C++编写的数据库访问代码：
```c++
  int Comm::setIDBySevNum(const XString& servnumber) {
    DB db;
    db.setSQL("select id from users where servnumber=:servnumber");
    db.bind(":servnumber", servnumber.c_str());
    db.open();

    if (!db.next()) {
      return -1;
    }

    setID(db.getString(”id"));
    return 0;
  }
```

它告诉我们，如果找不到需要的值，那么操作失败，返回-1，否则，返回0，成功了。

显然，写下这段代码的人有着C语言的背景，因为在C语言里面，我们常常会用整数表示成功失败。我说过，这是一段C++代码，而C++里面有一种类型叫做bool。

整数之所以能够占有本该属于布尔类型的舞台，很大程度上是受到C语言本身的限制。当然，C99之后，C程序员们终于有了属于自己的体面的布尔类型。

只是还有为数不少的C程序员依然生活在那个蛮荒年代。于是，很多人通过各种不尽如人意的方式模拟着布尔类型。不过，我们也看到了，偏偏就有这些生在福中不知福的程序员努力的重现着旧日时光。在我的职业生涯中，我见过许多用不同语法编写的C程序。

就个人学习语言经验而言，了解了基本的语法之后，如果有可能，我希望找到一本 Effective，寻求这门语言的编程之道。很多语言都有着自己的Effective，比如《Effective C++》、《Effective Java》、《Effective C#》，等等。

不了解语言，也会给丑陋代码可乘之机。比如，下面这段C++代码；
```c++
  void CommCode::notifyCRM(XString* retparam) {
    if (NULL == retparam) {
      throw IllegalArgumentsException(GetErrorMsg(" CommCode ::notifyCRM"));
    }
    ...
  }
```

如果把指针换成引用，就可以省去参数为空的判断，因为在C++里，引用不为空。这里选择了一个简单的例子，而在真实的代码里，这种检查漫天遍野，其丑陋可想而知。某些函数里面，检查甚至超过了真正的执行部分。

工欲善其事，必先利其器。有了铲子，就别再用手挖地了。