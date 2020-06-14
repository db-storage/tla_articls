# 用一个状态机描述整个分布式系统？谈谈Lamport被误读的"Time, Clock"一文以及状态机

# 1. 概要

在阅读Lamport的Paper、学习TLA+形式化验证过程中，发现状态机贯穿了Lamport的大量研究。例如，Paxos/FastPaxos的归纳法推导基于状态机、不变式和强归纳法；TLA+的验证基于状态机和 Model Checker运行过程中对不变式的验证，"Distributed Snapshot"一文，也基于状态机。但是大部分人读者都忽略了状态机的重要性，更没有注意到，Lamport的分布式理论的基础，是把**整个分布式系统，描述成了一个状态机**。如果你觉得分布式系统应该是N个状态机而不是一个状态机，那么你应该

Lamport在自己的主页上说，"Time, Clocks and the Ordering of Events in a Distributed System" [1]是他被引用最多的文章，但是**他几乎没遇到谁明白这篇文章是在写状态机**，而他写此文恰恰就是为了探讨分布式状态机。[原文链接](http://lamport.azurewebsites.net/pubs/pubs.html#time-clocks)

>It didn't take me long to realize that an algorithm for totally ordering events could be used to implement any distributed system. A distributed system can be described as a particular sequential state machine that is implemented with a network of processors. The ability to totally order the input requests leads immediately to an algorithm to implement an arbitrary state machine by a network of processors, and hence to implement any distributed system. So, **I wrote this paper, which is about how to implement an arbitrary distributed state machine.** As an illustration, I used the simplest example of a distributed system I could think of--a distributed mutual exclusion algorithm. 
>
>This is my most often cited paper. Many computer scientists claim to have read it. **But I have rarely encountered anyone who was aware that the paper said anything about state machines.** People seem to think that it is about either the causality relation on events in a distributed system, or the distributed mutual exclusion problem. People have insisted that there is nothing about state machines in the paper. I've even had to go back and reread it to convince myself that I really did remember what I had written. 

 本文主要包括两个部分：

- 讨论状态机、不变式及其在分布式系统正确性验证、证明方面的应用，这些内容主要来自于[2]；
- 讨论为什么能够把分布式系统用一个状态机来描述，内容主要来自于"Time, Clocks"一文；

个人对Lamport一些理论间的关系，总结如下：

[在这里画的图](https://app.lucidchart.com/documents/edit/191d2421-6e8c-4233-ab5d-563d1c65875d/0_0) (ppt)

 

For now, I take the simplest: a computation is a sequence of steps, which I call a behavior.



which views computations as partially ordered sets of events, is given in [7].





# 2. 为什么可以把整个分布式系统看成一个状态机？

## 2.1 状态机的串行性 vs 分布式系统的并行性



## 2.2 如何把分布式系统变成串行的状态机？ 

狭义相对论告诉我们，不存在一个不变的时空中事件的总顺序，不同的观察者可能就两个事件谁先发生产生分歧。但是部分顺序是存在的，如果e1可以因果影响e2，则事件e1早于事件e2(存在部分顺序)。

> Special relativity teaches us that there is no invariant total ordering of events in space-time; different observers can disagree about which of two events happened first. 
>
>  There is only a partial order in which an event e1 precedes an event e2 iff e1 can causally affect e2. 

### 2.2.1 一个生活中的例子：

假设小明和小华想把他们周末某一天的生活记录下来，做成一个短视频，放到某短视频平台上。他们各自用手机自拍了一些镜头，让你负责剪辑，做成一个短视频。你在剪辑过程中，哪些顺序是不能随便替换的？哪些是无所谓的？

假设短视频包括的画面如下：

1. 小明和小华早起后各自刷牙洗脸、下楼(假设他们住在不同的房子里，不用抢洗手间)；
2. 小明和小华一起坐西郊观光线去香山公园；
3. 他们分别公园门口买了一个山东煎饼了；
4. 他们在香炉峰、双清别墅各自拍照；
5. 还有些其他的零散的赏景镜头；
6. 他们一起坐西郊观光线回家；
7. 他们回家后各自发朋友圈。

如果不考虑倒叙，部分画面间是不能随便调整顺序的，部分是可以的。比如： 

- 小明和小华的刷牙顺序，哪个先放？ 无所谓，可以随便调整。
- 小明刷牙在小华洗脸，哪个先放？ 无所谓。
- 先放小华在公园拍照，然后再放小明在家洗脸，那就有点不合逻辑了，因为它们一起坐车去的公园。
- 先放小明在公园吃煎饼，再放他刷牙？ 不合逻辑。

​      

显然，你可以通过合理的剪辑，把小明和小华一天的生活，做成一个看起来不穿帮、合理的小视频。甚至本来小明刷牙更早，但是你剪辑后，小华刷牙的画面在前面，却没人发现。

明明是两个独立的人，看起来完全并发，为什么把他两的部分生活画面按照某个顺序剪辑出来，即使有些被点到顺序，看起来却没有穿帮，好像很合理呢？而有些画面顺序又不能调整？ 这就要回到 Lamport 的paper的关键概念： partial order 和 total order。      

### 2.2.2 偏序和全序

   Partial ordering 是指那些有逻辑依赖关系的事件之间的顺序，不能修改。这些事件是所有事件的一部分，所以称为Partial。另外，偏序关系具有非对称、传递性和反自反性。

而Total Ordering是指所有的Event之间都有一个顺序关系。在分布式系统中，这并不容易，所以才有了Lamport的"Time, Clock"一文。



比如，小王在观看这个纪录片时，看到了小华洗脸发生在小明洗脸之前，但是事实上不一定如此，这依赖于如何剪辑影片。不过这个无所谓，因为我们最终呈现的是一个确定的合乎逻辑的一幕幕即可。

​     我们把最终剪辑的短视频中的每一幕看成一个 event，那么小王看的这个短视频，每一幕之间实际上都是有序的，这个顺序就是一个total ordering。有逻辑依赖关系的event之间的顺序，必须在 total ordering 中被保持。

   不过这里有个要求，即不能把物理时间在每一幕都展示，如果每一幕都带着物理时间，那么可能就出现另外一个问题了。剪辑时得非常小心，免得让小王感觉时钟一会正着走，一会反着走。

   为什么有些画面可以调整顺序呢？因为它们之间没有依赖关系。比如，小明在8:00给小华打电话说要出门。加入打电话前，两人都刷牙洗脸了，那么到底小明刷牙早，还是小华刷牙早，已经无所谓。

 

为什么有些画面不能调整顺序？因为有依赖关系，这种依赖关系有两种：

- **每个人自己完成的事情间的相互顺序**，比如，小明先刷牙后洗脸，然后打电话约小华，游览完成才发朋友圈。如果纪录片中小明先发朋友圈，后坐车去公园，就乱套了。
- 不同的人，所做的事情间的顺序也有些顺序，这些顺序是因为他们之间的交互引起的。比如，小明跟小华一起坐车去公园这一刻，就是一个明确的时间点，它让二者之间建立了一个明确的联系，小明在公园拍照，不可能在小华刷牙之前，因为二者一起坐车去的公园。     小华刷牙 -> 二人坐车去公园 -> 小明在公园拍照。所以有 小华刷牙     -> 小明在公园拍照。

 

1. Partial Ordering只有唯一的一个，是确定的。

1. Total Ordering可以有多种，但是可能我们可以选择按照其中一种执行。
2. 对于形式化验证，可能要穷举所有可能的Total Order。

  

可能有大量的状态序列(每个序列有大量的状态)

 

为什么需要用状态序列去描述？

- 可以用数学去描述过程，以及可以做适当的变化，例如分布式快照一文正确性证明过程，就依赖于可以将一个状态序列S1，经过不违背Partial     Ordering的变换，变成快照状态S2，从而证明快照是个可达的合法状态，即使不是真的在某个时刻存在过；
- 基于不变式的归纳法证明，也是基于状态机理论。
- 在做形式化验证时，可以穷举所有可能的状态序列，去验证每个状态序列是否都是允许出现的，从而验证算法或者协议的正确性。例如，验证cache算法会不会出现同一个同一个数据的两份不同的cache，违背了一致性。如果没有状态序列的概念，完全看成杂乱无章的变化，连穷举都不可能。

  

### 2.2.3 怎么确定一个total order?

- 如果有logical    clock，可以让logical time相同的event，根据host id来排序下。



# 3. 不变式和归纳法





[1]  **Time, Clocks and the Ordering of Events in a Distributed System**, Lamport, *Communications of the ACM 21*, 7  (July 1978), 558-565

[2]**Computation and State Machines**,  https://lamport.azurewebsites.net/pubs/state-machine.pdf
