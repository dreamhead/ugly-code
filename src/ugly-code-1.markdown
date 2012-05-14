代码之丑（一）
===

诸位看官，上代码：
```c++
  if (0 == iRetCode) {
    this->SendPeerMsg("000", "Process Success", outRSet);
  } else {
    this->SendPeerMsg("000", "Process Failure", outRSet);
  }
```

乍一看，这段代码还算比较简短。那下面这段呢？
```c++
  if(!strcmp(pRec->GetRecType(), PUB_RECTYPE::G_INSTALL)) {
    CommDM.jkjtVPDNResOperChangGroupInfo(
      const_cast(CommDM.GetProdAttrVal("vpdnIPAddress",
      &(pGroupSubs->m_ProdAttr))),
      true);
  } else {
    CommDM.jkjtVPDNResOperChangGroupInfo(
      const_cast(CommDM.GetProdAttrVal("vpdnIPAddress",
      &(pGroupSubs->m_ProdAttr))),
      false);
  }
```

看出来问题了吗？经过仔细的对比，我们发现，对于如此华丽的代码，if/else的执行语句真正的差异只在于一个参数。第一段代码，二者的差异只是发送的消息，第二段代码，差异在于最后那个参数。

看破这个差异之后，新的写法就呼之欲出了，以第一段代码为例：
```c++
  const char* msg = (0 == iRetCode ? "Process Success" : "Process Failure");
  this->SendPeerMsg("000", msg, outRSet);
```

为了节省篇幅，我选择了条件表达式。我知道，很多人不是那么喜欢它。如果if/else依旧是你的大爱，勇敢追求去吧！

由这段代码调整过程，我们得出一个简单的规则：

* 让判断条件做真正的选择。

这里判断条件真正判断的内容是消息的内容，而不是消息发送的过程。经过我们的调整，得到消息内容和和发送消息的过程严格分离开来。

消除了代码中的冗余，代码也更容易理解，同时，给未来留出了可扩展性。如果将来iRetCode还有更多的情形，我们只要在消息获取的时候进行调整就好了。当然，封装成一个函数是一个更好的选择，这样代码就变成了：
```c++
  this->SendPeerMsg("000", peerMsgFromRetCode(iRetCode), outRSet);
```

至于第二段代码的调整，留给你练手了。

这样丑陋的代码是如何从众多代码中脱颖而出的呢？很简单，只要看到，if/else两个执行块里面的内容相差无几，需要我们人工比字符寻找差异，恭喜你，你找到它了。