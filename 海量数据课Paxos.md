title: 海量数据课
date: 2015-11-23 17:25:54
tags: 课程｜论文
----

> # Paxos Make Simple/Transparent

## 1.Paxos 算法流程：

1. 假设包含2个老板（Client），3个政府官员（Acceptor）。
2. 老板1（Client1）的议题为“A项目我要中标。”，然后命他的秘书Proposer1去找政府官员（Accepter）。
3. Proposer1小姐带着现金同时找到了Acceptor-1~Acceptor-3官员，1与2号官员分别收取了10比特币，找到第3号官员时，没想到遭到了3号官员的鄙视，3号官员告诉她，Proposer-2给了11比特币。不过没关系，Proposer1已经得到了1,2两个官员的认可，形成了多数派(如果没有形成多数派，Proposer1会去银行提款在来找官员们给每人20比特币，这个过程一直重复每次+10比特币，直到多数派的形成)，满意的找老板复命去了，但是此时Proposer2保镖找到了1,2号官员，分别给了他们11比特币，1,2号官员的态度立刻转变，都说Proposer2的老板懂事，这下子Proposer2放心了，搞定了3个官员，找老板复命去了，当然这个过程是第一阶段提交，只是官员们初步接受贿赂而已。故事中的比特币是编号，议题是value。
4. 现在进入第二阶段提交，现在proposer1小姐使用分身术(多线程并发)分了3个自己分别去找3位官员，最先找到了1号官员签合同，遭到了1号官员的鄙视，1号官员告诉他proposer2先生给了他11比特币，因为上一条规则的性质proposer1小姐知道proposer2第一阶段在她之后又形成了多数派(至少有2位官员的赃款被更新了);此时她赶紧去提款准备重新贿赂这3个官员(重新进入第一阶段)，每人20比特币。刚给1号官员20比特币， 1号官员很高兴初步接受了议题，还没来得及见到2,3号官员的时候，在这时proposer2先生也使用分身术分别找3位官员(注意这里是proposer2的第二阶段)，被第1号官员拒绝了告诉他收到了20比特币，第2,3号官员顺利签了合同，这时2，3号官员记录client2老板用了11比特币中标，因为形成了多数派，所以最终接受了Client2老板中标这个议题，对于proposer2先生已经出色的完成了工作。
5. proposer-1小姐找到了2号官员，官员告诉她合同已经签了，将合同给她看，proposer-1小姐是一个没有什么职业操守的聪明人，觉得跟Client1老板混没什么前途，所以将自己的议题修改为“Client2老板中标”，并且给了2号官员20比特币，这样形成了一个多数派。顺利的再次进入第二阶段。由于此时没有人竞争了，顺利的找3位官员签合同，3位官员看到议题与上次一次的合同是一致的，所以最终接受了，形成了多数派，proposer-1小姐跳槽到Client2老板的公司去了。
以上Paxos的一致性得以保证。

### 注：关于提议ID的声称算法如下：
假设有3个proposer P1、P2、P3，开始m=0,编号分别为0，1，2。

1. P1提交的时候发现了P2已经提交，P2编号为1 > P1的0，因此P1重新计算编号：new P1 = 1*3+0 = 4（此时，P1编号相当于上述例子的20比特币；P2编号相当于11比特币。）
2. P3以编号2提交，发现小于P1的4，因此P3重新编号：new P3 = 1*3+2 = 5。

### 链接：
[Paxos算法细节详解(一)--通过现实世界描述算法](http://www.cnblogs.com/endsock/p/3480093.html)

## 2.State Machine Replication

1.fault tolerance技术.在这个模型中，程序（比如一个apache server）被视为 deterministic state machine，意思就是给程序一定顺序的 input requests，程序执行后就会到达一定的状态（准确的说是数据结果），而replication就是在多个 nodes 中保持相同的state。当然这要求我们的SMR模型可以容忍来自node和network的failures，否则仅是理论上做到.

2.给予每个replicas的input request sequence一致，在deterministic execution的前提下，这些replicas就会 reach the same exact state.

3.DMT技术。如果给予每个replica nodes相同的input，在DMT的作用下，可以使得这些input的线程调度在每个nodes上都是一样的，这样就保证了即使在每个nodes上是多线程执行input，这些nodes最终的状态也是一致的，因为它们的线程调度是一致的。（归功于一个logical time在schedule它们，这个time是根据代码运行的先后来的，overhead不高，约为12.7%。）

4.checkpoint mechanism。这需要使用CRIU来实现。

### 问题
问题一：什么情况下会令secondary节点忽略来自leader的提案？ 

问题二：为什么一个提案要获得大多数认可才能提交？

### 链接:
[State Machine Replication 技术 和 PAXOS 算法](http://m.blog.csdn.net/blog/BlueCloudMatrix/44239983)
