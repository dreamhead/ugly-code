代码之丑（五）
===

不知道为什么，初见它时，我想起了郭芙蓉的排山倒海：
```c++
  ColdRule *newRule = new ColdRule();
  newRule->SetOID(oldRule->GetOID());
  newRule->SetRegion(oldRule->GetRegion());
  newRule->SetRebateRuleID(oldRule->GetRebateRuleID());
  newRule->SetBeginCycle(oldRule->GetBeginCycle() + 1);
  newRule->SetEndCycle(oldRule->GetEndCycle());
  newRule->SetMainAcctAmount(oldRule->GetMainAcctAmount());
  newRule->SetGiftAcctAmount(oldRule->GetGiftAcctAmount());
  newRule->SetValidDays(0);
  newRule->SetGiftAcct(oldRule->GetGiftAcct());
  rules->Add(newRule);
```

就在我以为这一片代码就是完成给一个变量设值的时候，突然，在那个不起眼的角落里，这个变量得到了应用：它被加到了rules里面。什么叫峰回路转，这就是。

既然它给了我们这么有趣的体验，必然先杀后快。下面重构了这个函数：
```c++
  ColdRule* CreateNewRule(ColdRule& oldRule) {
    ColdRule *newRule = new ColdRule();
    newRule->SetOID(oldRule.GetOID());
    newRule->SetRegion(oldRule.GetRegion());
    newRule->SetRebateRuleID(oldRule.GetRebateRuleID());
    newRule->SetBeginCycle(oldRule.GetBeginCycle() + 1);
    newRule->SetEndCycle(oldRule.GetEndCycle());
    newRule->SetMainAcctAmount(oldRule.GetMainAcctAmount());
    newRule->SetGiftAcctAmount(oldRule.GetGiftAcctAmount());
    newRule->SetValidDays(0);
    newRule->SetGiftAcct(oldRule.GetGiftAcct());
    return newRule
  }

  rules->Add(CreateNewRule(*oldRule));
```

把这一堆设值操作提取了出来，整个函数看上去一下子就清爽了。不是因为代码变少了，而是因为代码按照它职责进行了划分：创建的归创建，加入的归加入。之前的代码之所以让我不安，多重职责是罪魁祸首。

谈论干净代码时，我们总会说，函数应该只做一件事。函数做的事越多，就会越冗长。也就越难发现不同函数内存在的相似之处。为了一个问题，要在不同的地方改来改去也就难以避免了。

即便大家都认同了函数应该只做一件事，但多大的一件事算是一件事呢！不同的人心里有不同的标准。有人甚至认为一个功能就是一件事。于是，代码会越来越刺激。

想写干净代码，就别怕事小。哪怕一个函数只有一行，只要它能完整的表达一件事。在干净代码的世界里，大心脏是不受喜欢的。

接下来，我需要用历经沧桑的口吻告诉你，这么跌宕起伏的代码也只不过是一个更大函数的一个部分。此刻，浮现在我脑海里的是层峦叠嶂的山峰。