---
layout: default
title: "为什么说Clojure是一门好语言"
nav_order: {{ page.date }}
---
Clojure是Rich Hickey发明的Lisp方言。Clojure选择JVM或CLR作为其运行环境。Clojure代码在运行前会被编译成Java字节码，因此Clojure可以很方便地调用Java代码，且具有不错的运行速度。了解过Lisp的人都对高阶函数，宏，laziness，map等语言特性着迷不已，但是由于其性能不足等原因，在实际生产中却很少见到Lisp的身影。Clojure是一种可以广泛用于实际生产的Lisp方言，除了性能之外Clojure还有很多其它优点。

# immutable和mutable的平衡，支持代码并行化
函数式编程强调编写没有副作用和状态的代码，代码的输出只决定于输入，而与某个全局变量或类成员当时的状态无关。这样的代码易于测试和改写，而不容易出错。但是在编写软件时，严格的没有状态的代码却很难编写。Clojure在不变性和可变性之间做了很好的折中。普通变量都是不可变的，只有特殊变量是可变的。这些特殊变量是atom，ref，agent。在编写Clojure代码时，可变状态只被用在必须的部分，从而最小化非函数式代码。函数式的代码不仅不易出错，而且很容易并行化。举个例子，有一个数的list，需要我们去计算其中每一个数的n次方。可能一开始我们只使用一个串行的map运算，如果有一天数组变得很大，我们希望加快计算的速度，由于n次方计算是一个函数，我们可以使用一个并行化的map版本，并行化的map函数会使用多个行程对这个list同时求值。大家可能觉得n次方太特殊了，我们日常的代码难以写成函数式，其实所有代码都可以转换成函数式，很多情况下我们只需要多做几次拷贝就可以了。函数式的代码特别适合并行化计算，因为除了输入外决定结果的其它因素都是不变的，所以我们可以安全地将串行代码改成并行计算，这就特别适合当今多核计算机结构。Clojure还有一个async库，其支持csp模型，我们可以像写串行代码一样实现异步行的代码。有了async库的帮助，我们可以更容易实现并行执行的程序。总之Clojure非常适合编写并行化代码。

# 高效的数据结构
Clojure不仅支持list，还支持hashmap，vector, set等集合数据结构。在Python和Perl里也有类似的数据结构。日常的数据基本都可以用这些数据结构来组织，且这三者具有类似的协议，因此它们都可以作为map，reduce，doseq等操作的对象。有了map等遍历数据结构的方式，我们可以方便地遍历各种集合。这样代码会更加紧凑，且编译器可以根据不同数据结构生成更加高效的目标代码。根据我自己的经验，不用使用index变量或遍历指针，我可以将注意力放在编程任务本身，编码效率更高。根据Channel 9的Brain Beckman，世界上有两类编程语言，一类是由上而下的，另一类是由下而上的。所谓由上而下和由下而上指的是代码实现的方式。使用由上而下的编程语言时，编程是一个逐渐细化的过程，而使用由下而上的编程语言时，注意力会先关注在变量，循环，判断这些底层元素上，再逐渐转移到高层的结构上。Clojure就是一种由上而下的语言，这更贴近人类思考的习惯。Rich Hickey 和 Brian Beckman 有一个[Inside Clojure][rich-brian-talk]的访谈(英文)，强烈建议学习Clojure的朋友观看。

# Web开发和ClojureScript
Clojure和Java一样都对Web 开发有很好的支持，用Clojure开发网络应用非常方便，只需很少的代码就可以完成一个应用，因为Clojure有丰富的为web服务的工具库。除去后端开发我们还将Clojure应用于前端开发，这就是ClojureScript。ClojureScript会被编译成JavaScript运行在JS虚拟机上。有了ClojureScript就可以前后端都用Clojure作为开发语言。使用Clojure开发前段，克服了JavaScript不易维护的缺点，且可以直接应用函数式编程，这也正是JavaScript倡导的编程风格。ClojureScript还支持React Native，进而用于app开发。ClojureScript扩大了Clojure的应用领域，让后端程序员可以兼顾前端开发工作。

# 活跃的开源社区
Clojure还是一个开源项目。有很多优秀程序员为Clojure贡献了大量有用的开源项目，而且这个数目还在不断增加中。这些项目大多托管于github，任何人都可以使用和贡献代码。Clojure社区的主要推动者Cognitect公司会举办很多关于Clojure的会议，虽然大多在美国和欧洲，但是演讲都可以在网上找到，其中有很多演讲对学习者很有帮助。学习者可以方便地找到所需的文档资料。

## _参考条目_
* [Cognitect公司主页][cognitect-site]
* [Clojure语言主页][clojure-site]
* [Clojure 社会化协作文档][clojure-docs]
* [Clojure 公案][clojure-koans]
* [Clojure Github Repository][clojure-github]
* [ClojureScript Github Repository][clojurescript-github]
* [Inside Clojure (Rich Hickey & Brian Beckman)][rich-brian-talk]

[cognitect-site]: http://www.cognitect.com/
[clojure-site]: http://www.clojure.org/
[clojure-docs]: http://clojuredocs.org/
[clojure-koans]: http://clojurekoans.com/
[clojure-github]: https://github.com/clojure/clojure
[clojurescript-github]: https://github.com/clojure/clojurescript
[rich-brian-talk]: https://channel9.msdn.com/Shows/Going+Deep/Expert-to-Expert-Rich-Hickey-and-Brian-Beckman-Inside-Clojure
