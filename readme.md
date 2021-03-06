SCIP回顾与思考

## 前言
借着疫情在家的两个月，总算把《Structure and Interpertation of Computer Programs》(中译名：计算机程序的构造和解释，简称SICP)的坑填完了。由于
在家基本是看书五分钟，摸鱼两小时，究竟理解了多少，是要打上问号的。于是便有了写这篇文章的初衷，即内容的回顾与思考。

SICP全书分为五章，分别是
- 构造过程抽象
- 构造数据抽象
- 模块化、对象和状态
- 元语言抽象
- 寄存器机器里的计算
前两章弥漫着函数式编程的思想，彻底地打破了过程和数据的界限，过程即数据，数据即过程。
第三章引入了赋值，于是有了模块化和对象状态，并且通过延时求值构造了一个无限数据模型，也就是流。
第四章实现了一门编程语言，其中eval和apply的递归调用，清晰地展示过程的定义和调用的整个图景。第五章则实现了一个计算机和编译器。

总的来说，SICP是一本修炼内功的书。之前的编程学习就像在森林里走路，按照给定的规则去做，最后总能走出去的，而SICP更像是将一下子带到森林上方，
放眼望去路在哪里一目了然，后面怎么走就是一件相当自然的事了。

理解了SICP，就像入门了Minecraft，有能力的完全可以自己打造出一个编程世界了。



