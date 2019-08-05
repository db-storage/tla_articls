> 
>
> ![Lamport](http://lamport.azurewebsites.net/leslie.jpg)
>
> TLA+是Lamport大神近年来着力研究和推广的形式化描述语言，TLC Model Checker则是相应的形式化验证工具。
>
> Lamport是图灵奖获得者，早期著名的贡献包括Latex和Paxos等等等等。就不再赘述了。
>
> Paxos当年就以难于理解著称，以至于有后来有很多文章去解释它，Lamport自己写了一篇"Paxos Made Simple"，大家仍然觉得看不懂。Paxos还有很多变种(比如Raft)，但是万变不离其宗。
>
> TLA+相当抽象难懂，我本人也在不断学习中。想系统地介绍TLA+并让文档易于理解是个非常艰难的工作，因为它本身就比较偏理论，完全用数学来描述。其对算法的数学抽象，与大家平时接触的算法差别很大。而用于描述算法的TLA+也与大家学习的编程语言、伪代码等差别很大。
>
> 这个系列的文档将是个长期的业余工作，但是相信能够帮助大家解决一些复杂的问题。比如，设计了一个分布式系统或者并发算法后，即使运行了一段时间没发现问题，也不能确定真的没有Bug，是否所有的场景都测试到了。在这种情况下，用TLA+去描述算法，再用TLC Model Checker去做验证，就是非常有意义的。
>
> 另外，按照数学模型去思考，帮助我们理清设计某个特定算法过程中，需要保证什么，是非常有意义的。而不是仅仅关注算法流程本身。换句话说，从一开始就按照数学模型去思考和描述，可以产生对算法的更清晰的认识。
>
> 如果阅读或者实践中遇到问题，欢迎提问，以便于完善。

