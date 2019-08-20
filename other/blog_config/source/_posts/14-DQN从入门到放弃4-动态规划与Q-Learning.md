---
title: DQN从入门到放弃4-动态规划与Q-Learning
brief: 动态规划与Q-Learning
key: 'DQN'
tags:
  - 机器学习
toc: true
comments: true
date: 2019-08-20 15:25:39
---

![](/pic/11/bg.jpg)
价值函数与Bellman方程

<!-- more -->

## 1 上文回顾

在上一篇文章[DQN从入门到放弃 第三篇](https://zhuanlan.zhihu.com/p/21340755?refer=intelligentunit)中，我们分析到了Bellman方程，其方程
$$ v(s)=\mathbb E[R_{t+1}+\lambda v(S_{t+1})|S_t=s] $$

极其简洁，透出的含义就是价值函数的计算可以通过迭代的方式来实现。接下来本文将介绍如何构建基于Bellman方程的算法及Q-Learning。首先介绍动作价值函数

## 2 Action-Value function 动作价值函数

前面我们引出了价值函数，考虑到每个状态之后都有多种动作可以选择，每个动作之下的状态又多不一样，我们更关心在某个状态下的不同动作的价值。显然。如果知道了每个动作的价值，那么就可以选择价值最大的一个动作去执行了。这就是Action-Value function $ Q^\pi(s,a) $。那么同样的道理，也是使用reward来表示，只是这里的reward和之前的reward不一样，这里是执行完动作action之后得到的reward，之前state对应的reward则是多种动作对应的reward的期望值。显然，动作之后的reward更容易理解。

那么，有了上面的定义，动作价值函数就为如下表示：
\begin{align}
  Q^\pi(s,a)&=\mathbb E[r_{t+1}+\lambda r_{t+2}+\lambda ^2r_{t+3}+...+|s,a]\\\\
            &=\mathbb E_{s^\prime}[r \lambda Q^\pi(s^\prime,a^\prime)|s,a]
\end{align}


这里要说明的是动作价值函数的定义，加了$ \pi $,也就是说是在策略下的动作价值。因为对于每一个动作而已，都需要由策略根据当前的状态生成，因此必须有策略的支撑。而前面的价值函数则不一定依赖于策略。当然，如果定义$ v^\pi(s) $则表示在策略$ \pi $下的价值。

那么事实上我们会更多的使用动作价值函数而不是价值函数，因为动作价值函数更直观，更方便应用于算法当中。

##  3 Optimal value function 最优价值函数

能计算动作价值函数是不够的，因为我们需要的是最优策略，现在求解最优策略等价于求解最优的value function，找到了最优的value function，自然而然策略也就是找到。（当然，这只是求解最优策略的一种方法，也就是value-based approach，由于DQN就是value-based，因此这里只讲这部分，以后我们会看到还有policy-based和model-based方法。一个就是直接计算策略函数，一个是估计模型，也就是计算出状态转移函数，从而整个MDP过程得解）

这里以动作价值函数来分析。

首先是最优动作价值函数和一般的动作价值函数的关系：
$$ Q^*(s,a)=\max_\pi+Q^\pi(s,a) $$

也就是最优的动作价值函数就是所有策略下的动作价值函数的最大值。通过这样的定义就可以使最优的动作价值的唯一性，从而可以求解整个MDP。这部分在上一篇文章的评论中有介绍。

那么套用上一节得到的value function，可以得到
$$ Q^*(s,a)=\mathbb E_{s^\prime}[r+\lambda+\max+_{a^\prime}Q^*(s^\prime,a^\prime)|s,a] $$

因为最优的Q值必然为最大值，所以，等式右侧的Q值必然为使a′取最大的Q值。

下面介绍基于Bellman方程的两个最基本的算法，策略迭代和值迭代。

## 4 策略迭代Policy Iteration

Policy Iteration的目的是通过迭代计算value function 价值函数的方式来使policy收敛到最优。

Policy Iteration本质上就是直接使用Bellman方程而得到的：
$$
\begin{aligned} 
v_{k+1}(s) & \doteq \mathbb{E}_{\pi}\left[R_{t+1}+\gamma v_{k}\left(S_{t+1}\right) | S_{t}=s\right] \\\\ 
&=\sum_{a} \pi(a | s) \sum_{s^{\prime}, r} p\left(s^{\prime}, r | s, a\right)\left[r+\gamma v_{k}\left(s^{\prime}\right)\right] 
\end{aligned}
$$

那么Policy Iteration一般分成两步：

1.  Policy Evaluation 策略评估。目的是 更新Value Function，或者说更好的估计基于当前策略的价值
2.  Policy Improvement 策略改进。 使用 greedy policy 产生新的样本用于第一步的策略评估。
![](/pic/14/1.png)
本质上就是使用当前策略产生新的样本，然后使用新的样本更好的估计策略的价值，然后利用策略的价值更新策略，然后不断反复。理论可以证明最终策略将收敛到最优。
具体算法：
![](/pic/14/2.png)

## 5 Value Iteration 价值迭代

Value Iteration则是使用Bellman 最优方程得到
$$
\begin{aligned} 
v_{*}(s)  &=\max _{a} \mathbb{E}\left[R_{t+1}+\gamma v_{*}\left(S_{t+1}\right) | S_{t}=s, A_{t}=a\right] \\\\ 
          &=\max _{a} \sum_{s^{\prime}, r} p\left(s^{\prime}, r | s, a\right)\left[r+\gamma v_{*}\left(s^{\prime}\right)\right] 
\end{aligned}
$$

然后改变成迭代形式
$$
\begin{aligned} 
v_{*}(s)  &=\max _{a} \mathbb{E}\left[R_{t+1}+\gamma v_{*}\left(S_{t+1}\right) | S_{t}=s, A_{t}=a\right] \\\\ 
          &=\max _{a} \sum_{s^{\prime}, r} p\left(s^{\prime}, r | s, a\right)\left[r+\gamma v_{*}\left(s^{\prime}\right)\right] 
\end{aligned}
$$

value iteration的算法如下：
![](/pic/14/3.png)

那么问题来了：

*   Policy Iteration和Value Iteration有什么本质区别？
*   为什么一个叫policy iteration，一个叫value iteration呢？

原因其实很好理解，policy iteration使用bellman方程来更新value，最后收敛的value 即$ v_\pi $是当前policy下的value值（所以叫做对policy进行评估），目的是为了后面的policy improvement得到新的policy。

而value iteration是使用bellman 最优方程来更新value，最后收敛得到的value即$ v_* $就是当前state状态下的最优的value值。因此，只要最后收敛，那么最优的policy也就得到的。因此这个方法是基于更新value的，所以叫value iteration。

从上面的分析看，value iteration较之policy iteration更直接。不过问题也都是一样，需要知道状态转移函数p才能计算。本质上依赖于模型，而且理想条件下需要遍历所有的状态，这在稍微复杂一点的问题上就基本不可能了。

那么上面引用的是价值函数的版本，那么如果是使用动作价值函数呢，公式基本是一样的：
$$ Q_{i+1}(s,a)=\mathbb E_{s^\prime}[r+\lambda+\max_{a^\prime}Q_i(s^\prime,a^\prime)|s,a] $$

怎么直观的理解呢？

我们还在计算当前的Q值，怎么能有下一个Q值呢？没有错。所以，我们只能用之前的Q值。也就是没次根据新得到的reward和原来的Q值来更新现在的Q值。理论上可以证明这样的value iteration能够使Q值收敛到最优的action-value function。

## 6 Q-Learning

Q Learning的思想完全根据value iteration得到。但要明确一点是value iteration每次都对所有的Q值更新一遍，也就是所有的状态和动作。但事实上在实际情况下我们没办法遍历所有的状态，还有所有的动作，我们只能得到有限的系列样本。因此，只能使用有限的样本进行操作。那么，怎么处理？Q Learning提出了一种更新Q值的办法：
$$ Q(S_{t},A_{t})+\leftarrow+Q(S_{t},A_{t})+\alpha({R_{t+1}+\lambda+\max+_aQ(S_{t+1},a)}+-+Q(S_t,A_t)) $$

虽然根据value iteration计算出target Q值，但是这里并没有直接将这个Q值（是估计值）直接赋予新的Q，而是采用渐进的方式类似梯度下降，朝target迈近一小步，取决于α,这就能够减少估计误差造成的影响。类似随机梯度下降，最后可以收敛到最优的Q值。

具体的算法如下：
![](/pic/14/4.png)

### 7 Exploration and Exploitation 探索与利用

在上面的算法中，我们可以看到需要使用某一个policy来生成动作，也就是说这个policy不是优化的那个policy，所以Q-Learning算法叫做Off-policy的算法。另一方面，因为Q-Learning完全不考虑model模型也就是环境的具体情况，只考虑看到的环境及reward，因此是model-free的方法。 

回到policy的问题，那么要选择怎样的policy来生成action呢？有两种做法：

- 随机的生成一个动作

- 根据当前的Q值计算出一个最优的动作，这个policy$ \pi $称之为greedy policy贪婪策略。也就是
$$ \pi(S_{t+1})+=+arg\max+_aQ(S_{t+1},a) $$

使用随机的动作就是exploration，也就是探索未知的动作会产生的效果，有利于更新Q值，获得更好的policy。而使用greedy policy也就是target policy则是exploitation，利用policy，这个相对来说就不好更新出更好的Q值，但可以得到更好的测试效果用于判断算法是否有效。

将两者结合起来就是所谓的$ \epsilon-greedy $策略，$ \epsilon $一般是一个很小的值，作为选取随机动作的概率值。可以更改$ \epsilon $的值从而得到不同的exploration和exploitation的比例。

这里需要说明的一点是使用$ \epsilon-greedy $策略是一种极其简单粗暴的方法，对于一些复杂的任务采用这种方法来探索未知空间是不可取的。因此，最近有越来越多的方法来改进这种探索机制。

## 8 小结

到了这里，我们已经分析了Q-Learning算法，这也就是DQN所依赖的增强学习算法。下一步我们就讲直接分析DQN的算法实现了。

本文主要参考：

1 [Reinforcement Learning: An Introduction](https://link.zhihu.com/?target=https%3A//webdocs.cs.ualberta.ca/~sutton/book/the-book.html)
2 [Reinforcement Learning Course by David Silver](https://link.zhihu.com/?target=http%3A//www0.cs.ucl.ac.uk/staff/D.Silver/web/Teaching.html)

图片引用自：

[Reinforcement Learning Course by David Silver](https://link.zhihu.com/?target=http%3A//www0.cs.ucl.ac.uk/staff/D.Silver/web/Teaching.html) 的ppt


## 链接
[DQN 从入门到放弃1 DQN与增强学习](https://zhuanlan.zhihu.com/p/21262246?refer=intelligentunit)
[DQN 从入门到放弃2 增强学习与MDP](https://zhuanlan.zhihu.com/p/21292697?refer=intelligentunit)
[DQN 从入门到放弃3 价值函数与Bellman方程](https://zhuanlan.zhihu.com/p/21340755?refer=intelligentunit)
[DQN 从入门到放弃4 动态规划与Q-Learning](https://zhuanlan.zhihu.com/p/21378532?refer=intelligentunit)
[DQN 从入门到放弃5 深度解读DQN算法](https://zhuanlan.zhihu.com/p/21421729?refer=intelligentunit)
[DQN 从入门到放弃6 DQN的各种改进](https://zhuanlan.zhihu.com/p/21547911?refer=intelligentunit)
[DQN 从入门到放弃7 连续控制DQN算法-NAF](https://zhuanlan.zhihu.com/p/21609472?refer=intelligentunit)

<span style="font-family: &quot;Helvetica Neue&quot;, Helvetica, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif;background-color: rgb(255, 255, 255);letter-spacing: 0.5px;font-size: 14px;color: rgb(136, 136, 136);">
来源：

[Flood Sung](https://www.zhihu.com/people/flood-sung/activities)
[智能单元](https://zhuanlan.zhihu.com/intelligentunit)

</span>



