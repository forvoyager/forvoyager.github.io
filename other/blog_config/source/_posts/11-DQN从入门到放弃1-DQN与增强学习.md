---
title: DQN从入门到放弃1-DQN与增强学习
brief: DQN与增强学习
key: 'DQN'
tags:
  - 机器学习
toc: true
comments: true
date: 2019-08-14 11:14:48
---

![](/pic/11/bg.jpg)
DQN与增强学习

<!-- more -->

## 1 前言

深度增强学习Deep Reinforcement Learning是将深度学习与增强学习结合起来从而实现从Perception感知到Action动作的端对端学习End-to-End Learning的一种全新的算法。简单的说，就是和人类一样，输入感知信息比如视觉，然后通过深度神经网络，直接输出动作，中间没有hand-crafted engineering的工作。深度增强学习具备使机器人实现真正完全自主的学习一种甚至多种技能的潜力。

虽然将深度学习和增强学习结合的想法在几年前就有人尝试，但真正成功的开端就是DeepMind在NIPS 2013上发表的**[Playing Atari with Deep Reinforcement Learning](https://link.zhihu.com/?target=http%3A//arxiv.org/abs/1312.5602)**一文，在该文中第一次提出Deep Reinforcement Learning 这个名称，并且提出DQN（Deep Q-Network）算法，实现从纯图像输入完全通过学习来玩Atari游戏的成果。之后DeepMind在Nature上发表了改进版的DQN文章**Human-level Control through Deep Reinforcement Learning**，引起了广泛的关注，Deep Reinfocement Learning 从此成为深度学习领域的前沿研究方向。
![](/pic/11/1.jpg)

而Hinton，Bengio及Lecun三位大神在Nature上发表的[Deep Learning综述](https://link.zhihu.com/?target=http%3A//www.cs.toronto.edu/~hinton/absps/NatureDeepReview.pdf)一文最后也将Deep Reinforcement Learning作为未来Deep Learning的发展方向。引用一下原文的说法：
> We expect much of the
> future progress in vision to come from systems that are trained end-to-end
> and combine ConvNets with RNNs that use reinforcement learning
> to decide where to look.

从上面的原文可见三位大神对于Deep Reinforcement Learning的期待。而显然这一年来的发展没有让大家失望，AlphaGo横空出世，将进一步推动Deep Reinforcement Learning的发展。

Deep Reinforcement Learning的重要关键在于其具备真正实现AI的潜力，它使得计算机能够完全通过自学来掌握一项任务，甚至超过人类的水平。也因此，DeepMind很早受到了Google等企业的关注。DeepMind 50多人的团队在2014年就被Google以4亿美元的价格收购。而15年12月份刚刚由Elon Musk牵头成立的OpenAI，则一开始就获得了10亿美元的投资，而OpenAI中的好几位成员都来自UC Berkerley的Pieter Abbeel团队。

Pieter Abbeel团队紧随DeepMind之后，采用基于引导式监督学习直接实现了机器人的End-to-End学习，其成果也引起了大量的媒体报道和广泛关注。去年的NIPS 2015 更是由Pieter Abbeel及DeepMind的David Silver联合组织了Deep Reinforcement Learning workshop。可以说，目前在Deep Reinforcement Learning取得开拓性进展的主要集中在DeepMind和UC Berkerley团队。

为了研究Deep Reinforcement Learning，DQN的学习是首当其冲的。只有真正理解了DQN算法，才能说对Deep Reinforcement Learning入门。要理解并掌握DQN算法，需要增强学习和深度学习的多方面知识，笔者在2014年底开始接触DQN，但由于对基础知识掌握不全，导致竟然花了近1年的时间才真正理解DQN的整个算法。因此，本专栏从今天开始推出 **DQN 从入门到放弃 系列**文章，意在通过对增强学习，深度学习等基础知识的讲解，以及基于Tensorflow的代码实现，使大家能够扎实地从零开始理解DQN，入门Deep Reinforcement Learning。本系列文章将以一周一篇的速度更新。另外要说明的一点是DQN已被Google申请专利，因此只能做研究用，不能商用。

## 2 预备条件

虽然说是从零开始，但是DQN毕竟也还属于深度学习领域的前沿算法，为了理解本系列的文章，知友们还是需要有一定的基础：

*   一定的概率论和线性代数基础（数学基础）
*   一定的Python编程基础（编程基础，后面的代码实现将完全基于Tensorflow实现）

考虑到目前理解深度学习的知友肯定比理解增强学习的知友多，并且专栏也在同步翻译CS231N的内容，本系列文章计划用极短的篇幅来介绍DQN所使用的深度学习知识，而用更多的篇幅介绍增强学习的知识。

如果知友们具备以上的基本预备条件，那么我们就可以开始DQN学习之旅了。

接下来本文将介绍增强学习的基础知识。

## 3 增强学习是什么

在人工智能领域，一般用**智能体Agent**来表示一个具备行为能力的物体，比如机器人，无人车，人等等。那么增强学习考虑的问题就是**智能体Agent**和**环境Environment**之间交互的任务。比如一个机械臂要拿起一个手机，那么机械臂周围的物体包括手机就是环境，机械臂通过外部的比如摄像头来感知环境，然后机械臂需要输出动作来实现拿起手机这个任务。再举玩游戏的例子，比如我们玩极品飞车游戏，我们只看到屏幕，这就是环境，然后我们输出动作（键盘操作）来控制车的运动。

那么，不管是什么样的任务，都包含了一系列的**动作Action**,**观察Observation**还有**反馈值Reward**。所谓的Reward就是Agent执行了动作与环境进行交互后，环境会发生变化，变化的好与坏就用Reward来表示。如上面的例子。如果机械臂离手机变近了，那么Reward就应该是正的，如果玩赛车游戏赛车越来越偏离跑道，那么Reward就是负的。接下来这里用了Observation观察一词而不是环境那是因为Agent不一定能得到环境的所有信息，比如机械臂上的摄像头就只能得到某个特定角度的画面。因此，只能用Observation来表示Agent获取的感知信息。
![](/pic/11/2.png)

那么知道了整个过程，任务的目标就出来了，那就是要能获取尽可能多的Reward。没有目标，控制也就无从谈起，因此，获取Reward就是一个量化的标准，Reward越多，就表示执行得越好。每个时间片，Agent都是根据当前的观察来确定下一步的动作。观察Observation的集合就作为Agent的所处的**状态State**，因此，**状态State**和**动作Action**存在映射关系，也就是一个state可以对应一个action，或者对应不同动作的概率（常常用概率来表示，概率最高的就是最值得执行的动作）。状态与动作的关系其实就是输入与输出的关系，而状态State到动作Action的过程就称之为一个**策略Policy，**一般用![[公式]](https://www.zhihu.com/equation?tex=%5Cpi+)表示，也就是需要找到以下关系：

$$ a=\pi(s) $$
或者
$$ \pi(a|s) $$

其中a是action，s是state。第一种是一一对应的表示，第二种是概率的表示。

> 增强学习的任务就是找到一个最优的策略Policy从而使Reward最多。

我们一开始并不知道最优的策略是什么，因此往往从随机的策略开始，使用随机的策略进行试验，就可以得到一系列的状态,动作和反馈：

$$ \lbrace s_1,a_1,r_1,s_2,a_2,r_2,...s_t,a_t,r_t \rbrace $$

这就是一系列的**样本Sample**。增强学习的算法就是需要根据这些样本来改进Policy，从而使得得到的样本中的Reward更好。由于这种让Reward越来越好的特性，所以这种算法就叫做增强学习Reinforcement Learning。

## 4 What's Next?

在下一篇文章中，笔者将和大家分享MDP马尔科夫决策过程的知识，这是构建增强学习算法的基础。敬请关注！

## 链接
[DQN 从入门到放弃1 DQN与增强学习](https://zhuanlan.zhihu.com/p/21262246?refer=intelligentunit)

<span style="font-family: &quot;Helvetica Neue&quot;, Helvetica, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif;background-color: rgb(255, 255, 255);letter-spacing: 0.5px;font-size: 14px;color: rgb(136, 136, 136);">
来源：

[Flood Sung](https://www.zhihu.com/people/flood-sung/activities)
[智能单元](https://zhuanlan.zhihu.com/intelligentunit)

</span>



