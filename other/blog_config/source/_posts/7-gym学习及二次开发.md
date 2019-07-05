---
title: 7.gym学习及二次开发
brief: 机器学习
key: ''
tags:
  - 机器学习
toc: true
comments: true
date: 2019-07-05 11:23:05
---

![](/pic/7/1.jpg)

<!-- more -->

工欲善其事必先利其器，借助一个好的开发平台，我们能够快速地将理论知识变成代码，从而实现和验证我们的一些想法。Openai的gym（[openai/gym](https://link.zhihu.com/?target=https%3A//github.com/openai/gym)）便是这样一个好的平台。之所以说它好，是因为它背后有一支强大的团队在支持，维护和更新，这保证了平台的可持续性。而且它是openai于2017年5月16号，也就是前天释放出来的roboschool（[openai/roboschool](https://link.zhihu.com/?target=https%3A//github.com/openai/roboschool)）的基础；它还可以与谷歌的tensorflow联合使用，等等。因此，我们在实战系列的第一讲中比较全面地介绍下gym。为方便交流，公布个qq交流群202570720。

本讲打算分3个小节对gym进行介绍。
> 第1小节：gym安装及简单的demo示例  
> 第2小节：深入剖析gym的环境构建  
> 第3小节：构建自己的gym环境  

## 第1小节 gym安装及简单的demo示例

该小节讲的是gym在ubuntu下的安装，其他系统可参考官网安装过程。

### 1.1 为了便于管理，需要先装anaconda

具体下载和安装步骤如下：
1.下载anaconda安装包，推荐利用清华镜像来下载，下载地址为：
[https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive 。  ](https://link.zhihu.com/?target=https%3A//mirrors.tuna.tsinghua.edu.cn/anaconda/archive%25E3%2580%2582%2520%25E6%2588%2591%25E8%25A3%2585%25E7%259A%2584%25E6%2598%25AFAnaconda3-4.3.0)

[我装的是Anaconda3-4.3.0](https://link.zhihu.com/?target=https%3A//mirrors.tuna.tsinghua.edu.cn/anaconda/archive%25E3%2580%2582%2520%25E6%2588%2591%25E8%25A3%2585%25E7%259A%2584%25E6%2598%25AFAnaconda3-4.3.0)版本。

2.安装anaconda。下载完成anaconda后，安装包会在Dowloads文件夹下，在终Ctrl+Alt+T打开终端）键入cd Downloads， 然后键入 bash Anaconda3_4.3.0-Linux-x86_64.sh(小技巧，键入bash an然后按Tab键，linux系统会自动补全后面的名字)

3.安装过程会询问你是否将路径安装到环境变量中，键入yes， 至此Anaconda安装完成。你会在目录/home/你的用户名文件夹下面看到anaconda3。关掉终端，再开一个，这样环境变量才起作用。

### 1.2 利用anaconda建一个虚拟环境
Anaconda创建虚拟环境的格式为：conda create –-name 你要创建的名字 python=版本号。
比如我创建的虚拟环境名字为gymlab(你可以用自己的环境名）, 用的python版本号为3.5，可这样写：
```
conda create –-name gymlab python=3.5
```

操作完此步之后，会在anaconda3/envs文件夹下多一个gymlab。Python3.5就在gymlab下得lib文件夹中。

### 1.3 安装gym

上一步已经装了一个虚拟环境gymlab, 在这一步要应用。

开一个新的终端，然后用命令source activate gymlab激活虚拟环境，然后再装gym。具体步骤如下：

1.键入git clone [openai/gym](https://link.zhihu.com/?target=https%3A//github.com/openai/gym.git)，将gym克隆到计算机中，如果你的计算机中没有安装git, 那么可以键入：
```
sudo apt install git
```
先安装git.

2.cd gym 进入gym文件夹

3.pip install –e ‘.[all]’进行完全安装。等待，会装一系列的库等，装完后可以将你的gym安装文件的目录写到环境变量中，一种方法是打开.bashrc文件，在末尾加入语句：
```
export PYTHONPATH=你的gym目录：$PYTHONPATH。
```
如果不出意外的话，你就可以开始享用gym了。如果报错可以先安装依赖项，键入命令，然后再按照Step3的命令安装。
```
sudo apt-get install -y python-numpy python-dev cmake zlib1g-dev libjpeg-dev xvfb libav-tools xorg-dev python-opengl libboost-all-dev libsdl2-dev swig
```

### 下面给出一个最简单的例子

1.开一个终端(ctr+alt+t)， 然后激活用anaconda建立的虚拟环境：
```
source activate gymlab
```

2.运行python
```
python
```

3.导入gym模块
```
import gym
```

4.创建一个小车倒立摆模型
```
env = gym.make(‘CartPole-v0’)
```

5.初始化环境
```
env.reset()
```

6.刷新当前环境，并显示
```
env.render()
```

通过这6步，我们可以看到一个小车倒立摆系统.如下图所示：
![](/pic/7/2.png)

## 第2小节 深入剖析gym环境构建

我们继续讲，从第1小节的尾巴开始。有三个重要的函数：
env =gym.make('CartPole-v0')
env.reset()
env.render()

第一个函数是创建环境，我们会在第3小节具体讲如何创建自己的环境，所以这个函数暂时不讲。
第二个函数env.reset()和第三个函数env.render()是每个环境文件都包含的函数。我们以cartpole为例，对这两个函数进行讲解。

Cartpole的环境文件在~你的gym目录/gym/envs/classic_control/cartpole.py.

该文件定义了一个CartPoleEnv的环境类，该类的成员函数有：seed(),
step(),reset()和render().
第1小节调用的就是CartPoleEnv的两个成员函数reset()和render()。下面，我们先讲讲这两个函数，再介绍step()函数

### 2.1 reset()函数详解

reset()为重新初始化函数。那么这个函数有什么用呢？

在强化学习算法中，智能体需要一次次地尝试，累积经验，然后从经验中学到好的动作。一次尝试我们称之为一条轨迹或一个episode.
每次尝试都要到达终止状态.
一次尝试结束后，智能体需要从头开始，这就需要智能体具有重新初始化的功能。函数reset()就是这个作用。

reset()的源代码为：
```python
def _reset()
self.state = self.np_random.uniform(low=-0.05, high=0.05, size=(4,))
self.steps_beyond_done = None
return np.array(self.state)
```
第一行代码是利用均匀随机分布初试化环境的状态。
第二行代码是设置当前步数为None
第3行，返回环境的初始化状态。

### 2.2 render()函数详解

render()函数在这里扮演图像引擎的角色。一个仿真环境必不可少的两部分是物理引擎和图像引擎。物理引擎模拟环境中物体的运动规律；图像引擎用来显示环境中的物体图像。其实，对于强化学习算法，该函数可以没有。但是，为了便于直观显示当前环境中物体的状态，图像引擎还是有必要的。另外，加入图像引擎可以方便我们调试代码。下面具体介绍gym如何利用图像引擎来创建图像。

我们直接看源代码：
```python
def _render(self, mode=’human’, close=False):

if close:
#省略，直接看关键代码部分
if self.viewer is None:

from gym.envs.classic_control
import rendering     

#这一句导入rendering模块，利用rendering模块中的画图函数进行图形的绘制

#如绘制600*400的窗口函数为：
self.viewer = rendering.Viewer(screen_width, screen_height)
#其中screen_width=600，screen_height=400

#创建小车的代码为：
l,r,t,b = -cartwidth/2, cartwidth/2, cartheight/2, -cartheight/2
axleoffset =cartheight/4.0
cart = rendering.FilledPolygon([(l,b), (l,t), (r,t), (r,b)])
```

其中rendering.FilledPolygon为填充一个矩形。

创建完cart的形状，接下来给cart添加平移属性和旋转属性。将车的位移设置到cart的平移属性中，cart就会根据系统的状态变化左右运动。具体代码解释，我已上传到github上面了，[gxnk/reinforcement-learning-code](https://link.zhihu.com/?target=https%3A//github.com/gxnk/reinforcement-learning-code)　。想深入了解的同学可去下载学习。

### 2.3 step()函数详解

该函数在仿真器中扮演物理引擎的角色。其输入是动作a，输出是：下一步状态，立即回报，是否终止，调试项。

该函数描述了智能体与环境交互的所有信息，是环境文件中最重要的函数。在该函数中，一般利用智能体的运动学模型和动力学模型计算下一步的状态和立即回报，并判断是否达到终止状态。

我们直接看源代码：
```python
def _step(self, action):

assert self.action_space.contains(action), "%r (%s) invalid"%(action, type(action))

state = self.state

x, x_dot, theta, theta_dot = state #系统的当前状态

force = self.force_mag if action==1 else -self.force_mag #输入动作，即作用到车上的力

costheta = math.cos(theta) #余弦函数

sintheta = math.sin(theta) #正弦函数

#底下是车摆的动力学方程式，即加速度与动作之间的关系。

temp = (force + self.polemass_length * theta_dot * theta_dot * sintheta) / self.total_mass

thetaacc = (self.gravity * sintheta - costheta* temp) / (self.length * (4.0/3.0 - self.masspole * costheta * costheta / self.total_mass)) #摆的角加速度

xacc = temp - self.polemass_length * thetaacc * costheta / self.total_mass #小车的平移加速

x = x + self.tau * x_dot

x_dot = x_dot + self.tau * xacc

theta = theta + self.tau * theta_dot

theta_dot = theta_dot + self.tau * thetaacc #积分求下一步的状态

self.state = (x,x_dot,theta,theta_dot)                              
```

### 2.4 一个简单的demo
下面，我给出一个最简单的demo，让大家体会一下上面三个函数如何使用。
```python
import gym
import time
env = gym.make('CartPole-v0')   #创造环境
observation = env.reset()       #初始化环境，observation为环境状态
count = 0
for t in range(100):
    action = env.action_space.sample()  #随机采样动作
    observation, reward, done, info = env.step(action)  #与环境交互，获得下一步的时刻
    if done:             
        break
    env.render()         #绘制场景
    count+=1
    time.sleep(0.2)      #每次等待0.2s
print(count)             #打印该次尝试的步数
```

## 第3小节 创建自己的gym环境并利示例qlearning的方法

在上一小节中以cartpole为例子深入剖析了gym环境文件的重要组成。我们知道，一个gym环境最少的组成需要包括reset()函数和step()函数。当然，图像显示函数render()一般也是需要的。这一节，我会以机器人找金币为例给大家演示如何构建一个全新的gym环境，并以此环境为例，示例最经典的强化学习算法qlearning算法。在3.1节中，给出机器人找金币的问题陈述；第3.2节中，给出构建gym环境的过程；第3.3节中，利用qlearning方法实现机器人找金币的智能决策。全部代码已传到github上。

### 3.1 机器人找金币的问题陈述
![图1.1 机器人找金币](/pic/7/3.png)

上为机器人在网格世界找金币的示意图。该网格世界一共有８个状态，其中状态６和状态8为死亡区域，状态７为金币区域。机器人的初始位置为网格世界中任意一个状态。机器人从初始状态出发寻找金币。机器人进行一次探索，进入死亡区域或找到金币，本次探测结束。机器人找到金币的回报为１，进入死亡区域回报为－１，机器人在区域１－５之间转换时，回报为０。我们的目标是找到一个策略使得机器人不管处在什么状态（１－５）都能找到金币。对于这个机器人找金币的游戏，我们可以利用强化学习的方法来实现。

### 3.2 构建网格世界的gym环境

一个gym的环境文件，其主体是个类，在这里我们定义类名为：GridEnv, 其初始化为环境的基本参数，因为机器人找金币的过程是一个马尔科夫过程，我们在强化学习入门课程的第一讲已经介绍过了一个马尔科夫过程应该包括状态空间，动作空间，回报函数，状态转移概率。因此，我们在类GridEnv的初始化时便给出了相应的定义。网格世界的全部代码在[gxnk/reinforcement-learning-code](https://link.zhihu.com/?target=https%3A//github.com/gxnk/reinforcement-learning-code),文件名为 grid_mdp.py 我们看源代码：

状态空间为：
```python
self.states = [1,2,3,4,5,6,7,8]
```

动作空间为：
```python
self.actions = [**'n'**,**'e'**,**'s'**,**'w'**]
```

回报函数为：
```python
self.rewards = dict(); #回报的数据结构为字典
self.rewards['1_s'] = -1.0
self.rewards['3_s'] = 1.0
self.rewards['5_s'] = -1.0
```

状态转移概率为：
```python
self.t = dict(); #状态转移的数据格式为字典
self.t[**'1_s'**] = 6
self.t[**'1_e'**] = 2
self.t[**'2_w'**] = 1
self.t[**'2_e'**] = 3
self.t[**'3_s'**] = 7
self.t[**'3_w'**] = 2
self.t[**'3_e'**] = 4
self.t[**'4_w'**] = 3
self.t[**'4_e'**] = 5
self.t[**'5_s'**] = 8
self.t[**'5_w'**] = 4
```

有了状态空间，动作空间和状态转移概率，我们便可以写step(a)函数了。这里特别注意的是，step()函数的输入是动作，输出为：下一个时刻的动作，回报，是否终止，调试信息。尤其需要注意的是输出的顺序不要弄错了。对于调试信息，可以为空，但不能缺少，否则会报错，常用{}来代替。我们看源代码：
```python
def _step(self, action):
  
  #系统当前状态
  state = self.state 
  
  ＃判断系统当前状态是否为终止状态
  if state in self.terminate_states:
    return state, 0, True, {}

  key = "%d_%s"%(state, action) #将状态和动作组成字典的键值

  #状态转移
  if key in self.t:
    next_state = self.t[key]
  else:
    next_state = state
    self.state = next_state
    is_terminal = False
  if next_state in self.terminate_states:
    is_terminal = True
  if key not in self.rewards:
    r = 0.0
  else:
    r = self.rewards[key]
  return next_state, r,is_terminal,{}
```

step()函数就是这么简单。下面我们重点介绍下如何写render()函数。从图1.1机器人找金币的示意图我们可以看到，网格世界是由一些线和圆组成的。因此，我们可以调用rendering中的画图函数来绘制这些图像。

整个图像是一个600*400的窗口，可用如下代码实现：
```python
from gym.envs.classic_control import rendering

self.viewer = rendering.Viewer(screen_width, screen_height)

# 创建网格世界，一共包括11条直线，事先算好每条直线的起点和终点坐标，然后绘制这些直线，代码如下：
self.line1 = rendering.Line((100,300),(500,300))
self.line2 = rendering.Line((100, 200), (500, 200))
self.line3 = rendering.Line((100, 300), (100, 100))
self.line4 = rendering.Line((180, 300), (180, 100))
self.line5 = rendering.Line((260, 300), (260, 100))
self.line6 = rendering.Line((340, 300), (340, 100))
self.line7 = rendering.Line((420, 300), (420, 100))
self.line8 = rendering.Line((500, 300), (500, 100))
self.line9 = rendering.Line((100, 100), (180, 100))
self.line10 = rendering.Line((260, 100), (340, 100))
self.line11 = rendering.Line((420, 100), (500, 100))

# 接下来，创建死亡区域，我们用黑色的圆圈代表死亡区域，源代码如下：
# 创建第一个骷髅
self.kulo1 = rendering.make_circle(40)
self.circletrans = rendering.Transform(translation=(140,150))
self.kulo1.add_attr(self.circletrans)
self.kulo1.set_color(0,0,0)
# 创建第二个骷髅
self.kulo2 = rendering.make_circle(40)
self.circletrans = rendering.Transform(translation=(460, 150))
self.kulo2.add_attr(self.circletrans)
self.kulo2.set_color(0, 0, 0)

# 创建金币区域，用金色的圆来表示：
# 创建金条
self.gold = rendering.make_circle(40)
self.circletrans = rendering.Transform(translation=(300, 150))
self.gold.add_attr(self.circletrans)
self.gold.set_color(1, 0.9, 0)

# 创建机器人，我们依然用圆来表示机器人，为了跟死亡区域和金币区域不同，我们可以设置不同的颜色：
# 创建机器人
self.robot= rendering.make_circle(30)
self.robotrans = rendering.Transform()
self.robot.add_attr(self.robotrans)
self.robot.set_color(0.8, 0.6, 0.4)

# 创建完之后，给11条直线设置颜色，并将这些创建的对象添加到几何中代码如下：
self.line1.set_color(0, 0, 0)
self.line2.set_color(0, 0, 0)
self.line3.set_color(0, 0, 0)
self.line4.set_color(0, 0, 0)
self.line5.set_color(0, 0, 0)
self.line6.set_color(0, 0, 0)
self.line7.set_color(0, 0, 0)
self.line8.set_color(0, 0, 0)
self.line9.set_color(0, 0, 0)
self.line10.set_color(0, 0, 0)
self.line11.set_color(0, 0, 0)
self.viewer.add_geom(self.line1)
self.viewer.add_geom(self.line2)
self.viewer.add_geom(self.line3)
self.viewer.add_geom(self.line4)
self.viewer.add_geom(self.line5)
self.viewer.add_geom(self.line6)
self.viewer.add_geom(self.line7)
self.viewer.add_geom(self.line8)
self.viewer.add_geom(self.line9)
self.viewer.add_geom(self.line10)
self.viewer.add_geom(self.line11)
self.viewer.add_geom(self.kulo1)
self.viewer.add_geom(self.kulo2)
self.viewer.add_geom(self.gold)
self.viewer.add_geom(self.robot)

# 接下来，开始设置机器人的位置。机器人的位置根据其当前所处的状态不同，所在的位置不同。我们事先计算出每个状态处机器人位置的中心坐标，并存储到两个向量中，并在类初始化中给出：
self.x=[140,220,300,380,460,140,300,460]
self.y=[250,250,250,250,250,150,150,150]

# 根据这两个向量和机器人当前的状态，我们就可以设置机器人当前的圆心坐标了即：
if self.state is None: return None

self.robotrans.set_translation(self.x[self.state-1], self.y[self.state- 1])

return self.viewer.render(return_rgb_array=mode == 'rgb_array')
```
以上便完成了render()函数的建立。

reset()函数的建立

reset()函数常常用随机的方法初始化机器人的状态，即：
```python
def _reset(self):
  self.state = self.states[int(random.random() * len(self.states))]
  return self.state
```

全部的代码请去github上下载学习。下面重点讲一讲如何将建好的环境进行注册，以便通过gym的标准形式进行调用。其实环境的注册很简单，只需要３步：

**第一步：**
将我们自己的环境文件（我创建的文件名为grid_mdp.py)拷贝到你的gym安装目录/gym/gym/envs/classic_control文件夹中。（拷贝在这个文件夹中因为要使用rendering模块。当然，也有其他办法。该方法不唯一）

**第二步：**
打开该文件夹（第一步中的文件夹）下的__init__.py文件，在文件末尾加入语句：
```
from gym.envs.classic_control.grid_mdp
import GridEnv
```

**第三步：**
进入文件夹你的gym安装目录/gym/gym/envs，打开该文件夹下的__init__.py文件，添加代码：
```python
register(
id='GridWorld-v0',
entry_point='gym.envs.classic_control:GridEnv',
max_episode_steps=200,
reward_threshold=100.0,
)
```

第一个参数id就是你调用gym.make(‘id’)时的id,　这个id你可以随便选取，我取的，名字是GridWorld-v0
第二个参数就是函数路口了。

后面的参数原则上来说可以不必要写。
经过以上三步，就完成了注册。
下面，我们给个简单的demo来测试下我们的环境的效果吧：

我们依然写个终端程序：
```python
source activate gymlab   
python
env = gym.make(**'GridWorld-v0'**)
env.reset()
env.render()
```

代码运行后会出现如图1.2所示的效果：
![图1.2 机器人找金币](/pic/7/4.png)

### 3.3 qlearning方法实现机器人找金币
![图1.2 机器人找金币](/pic/7/5.png)

在强化学习入门第四讲中[强化学习入门 第四讲 时间差分法（TD方法） - 知乎专栏](https://zhuanlan.zhihu.com/p/25913410)，已经给出了qlearning的伪代码，现在我们利用已经建立的环境用python对伪代码进行实现，源代码如下：
```python
def qlearning(num_iter1, alpha, epsilon):
    x = []
    y = []
    qfunc = dict()   #行为值函数为字典
    #初始化行为值函数为0
    for s in states:
        for a in actions:
            key = "%d_%s"%(s,a)
            qfunc[key] = 0.0
    for iter1 in range(num_iter1):
        x.append(iter1)
        y.append(compute_error(qfunc))

        #初始化初始状态
        s = grid.reset()
        a = actions[int(random.random()*len(actions))]
        t = False
        count = 0
        while False == t and count <100:
            key = "%d_%s"%(s, a)
            #与环境进行一次交互，从环境中得到新的状态及回报
            s1, r, t1, i =grid.step(a)
            key1 = ""
            #s1处的最大动作
            a1 = greedy(qfunc, s1)
            key1 = "%d_%s"%(s1, a1)
            #利用qlearning方法更新值函数
            qfunc[key] = qfunc[key] + alpha*(r + gamma * qfunc[key1]-qfunc[key])
            #转到下一个状态
            s = s1;
            a = epsilon_greedy(qfunc, s1, epsilon)
            count += 1
    plt.plot(x,y,"-.,",label ="q alpha=%2.1f epsilon=%2.1f"%(alpha,epsilon))
return qfunc
```

qlearning是异策略算法，其行动策略为epsilon-greedy策略，而评估策略为greedy策略。在代码中的体现为：
行动策略为：
```
a = epsilon_greedy(qfunc, s1, epsilon)
```

评估策略为：
```
a1 = greedy(qfunc, s1)
key1 = "%d_%s"%(s1, a1)

#利用qlearning方法更新值函数
qfunc[key] = qfunc[key] + alpha*(r + gamma * qfunc[key1]-qfunc[key])
```

上面的qun[key1]就是评估的贪婪策略。

上面的qlearning算法中的greedy()和epsilon_greedy在文件中都有定义，大家可登录到github下载下来学习,该部分代码在[gxnk/reinforcement-learning-code](https://link.zhihu.com/?target=https%3A//github.com/gxnk/reinforcement-learning-code)目录下文件名为qlearning.py中。

**最后我们给出一个机器人找金币学习和策略的例子。**

机器人利用qlearning的方法学习，然后打印学到的策略，再测试学到的策略。具体代码在[gxnk/reinforcement-learning-code](https://link.zhihu.com/?target=https%3A//github.com/gxnk/reinforcement-learning-code)目录下learning_and_test文件中。

其他相关的代码都可在我的github上下载[gxnk/reinforcement-learning-code](https://link.zhihu.com/?target=https%3A//github.com/gxnk/reinforcement-learning-code)

<span style="font-family: &quot;Helvetica Neue&quot;, Helvetica, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif;background-color: rgb(255, 255, 255);letter-spacing: 0.5px;font-size: 14px;color: rgb(136, 136, 136);">
来源：

[强化学习知识大讲堂](https://zhuanlan.zhihu.com/p/26985029)

</span>



