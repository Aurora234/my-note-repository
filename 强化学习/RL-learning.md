[【强化学习的数学原理】课程：从零开始到透彻理解（完结）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1sd4y167NS/?spm_id_from=333.337.search-card.all.click&vd_source=f3eda140312dabd61ae75324d7195430)

强化学习的数学原理

强化学习的最终目标就是**求解最优策略**

# 第一章 基本概念

- **State**：在强化学习（RL）中，**State（状态，记作 $s$ 或 $s_t$）** 是**对环境在某一时刻的完整信息描述**，它包含了**所有对未来决策有用的信息**，不丢失关键历史依赖。

- **State Space（状态空间，记作 S）** 是**环境所有可能出现的状态的集合**。
  状态空间 = **所有可能的 “快照” 的全集**，智能体**永远在这个空间里游走**

- Action：**Action（动作，记作a或 at）** 是**智能体（Agent）在某一状态下，对环境施加的控制决策**。它是**智能体唯一能主动改变环境的手段**。

- **Action Space（动作空间，记作 \**A\**）** 是**智能体在任意状态下，所有**合法、可执行**动作的集合。**不同状态的Action Space是不一样的

- **State Transition（状态转移）**：在强化学习中，指**智能体在当前状态$$s_t$$ 采取动作 $$a_t$$后，环境从当前状态变为**下一状态$$s_{t+1}$$的过程 。

  最核心的 RL 流程：$$\boldsymbol{s_t \xrightarrow{a_t} s_{t+1}}$$

  完整表达（带奖励）：$$(s_t,\ a_t) \rightarrow (s_{t+1},\ r_t)$$。rt：同时获得的奖励1. 
  
  1. 确定性转移（Deterministic）
  
     给定 $$s_t, a_t$$，**唯一确定**$$s_{t+1}$$
  
     $$p(s' \mid s,a) =
     \begin{cases}
     1, & s' = f(s,a) \\
     0, & \text{其他}
     \end{cases}$$
  
     例子：
  
     - 迷宫：在位置 (2,3) 向右走 → 一定到 (2,4)
     - 围棋：落子位置确定，棋盘一定变那样
     - 特点：做了动作，结果 100% 确定
  
  2. 随机性转移（Stochastic）
  
     同样$$s_t,a_t$$，**可能跳到多个不同$$s_{t+1}$$**，服从概率分布
  
     $$\sum_{s'} p(s' \mid s,a) = 1$$
  
     例子：
     
     - 摇骰子、抽卡游戏
     - 有风的迷宫（你想右走，可能被吹到右上）
     - 现实物理系统（噪声、扰动）
     - 特点：动作相同，结果可能不同，是真实世界的常态



---



- **Policy**： State、Action、State Transition 都是**环境**，而 **Policy 是智能体本身**。

  **Policy（策略，记作$$\pi$$）**

  = **智能体的行为函数 / 决策规则 / 大脑**

  = 给定**当前状态$$s$$**，告诉我**该做什么动作*$$a$$*

  （1）确定性策略 Deterministic Policy

  $$a = \pi(s)$$

  看到状态$$s$$，**固定输出一个动作$$a$$**。

  （2）随机性策略 Stochastic Policy

  $$\pi(a \mid s)$$

  看到状态 $$s$$，输出**选择每个动作 $$a$$ 的概率分布**。

  这是深度强化学习**最常用**的形式（用于探索）。

  MDP 完整闭环：

  $$\underbrace{s_t}_{\text{状态}}
  \xrightarrow{\underbrace{\pi(a_t|s_t)}_{\text{策略}}}
  \underbrace{a_t}_{\text{动作}}
  \xrightarrow{\underbrace{P(s_{t+1}|s_t,a_t)}_{\text{状态转移}}}
  \underbrace{s_{t+1}}_{\text{新状态}},\ \underbrace{r_t}_{\text{奖励}}$$

  **Policy** 是**唯一可控**的东西，其他都是环境给定

---



- **Reward（即时奖励，记作 $$r_t$$）**：是**环境在智能体执行动作后，给出的一个标量反馈信号**，用来**评价这一步动作的好坏**。
  由状态、动作、下一状态共同决定：$$r_t = R(s_t,\ a_t,\ s_{t+1})$$。或简化为：$$r_t = R(s_t,\ a_t)$$

  1. 它是**标量（Scalar）**

     只有大小，没有方向，就是一个**数字**：

     - 正数：好
     - 零：无所谓
     - 负数：坏 / 惩罚

  2. **即时性（Immediate）**

     rt 只评价**第 \**t\** 步**，不看长远。

     ⚠️ 这是 RL 最难的地方：**局部好≠全局好**。

  3. **环境给出，不可修改**

     - Reward 是**环境规则**的一部分

     - 智能体**只能接受，不能改变奖励机制**

     - 我们设计者**只能设计 Reward**

---

**Trajectory（轨迹）**

Trajectory = 智能体与环境交互的一整条历史序列。就是从**开始到结束**，状态、动作、奖励一步一步按时间排下来的**完整经历**。

数学表示：$$\tau = (s_0,\,a_0,\,r_0,\,s_1,\,a_1,\,r_1,\,\dots,\,s_T,\,a_T,\,r_T)$$

按时间展开就是：$$s_0 \xrightarrow{a_0} r_0,s_1 \xrightarrow{a_1} r_1,s_2 \xrightarrow{a_2} \dots \xrightarrow{a_T} r_T,s_{T+1}$$

- $$s_0$$：初始状态
- $$T$$：终止步（回合结束）
- 每一步都由 **状态→动作→奖励→新状态** 组成

---

**Return（回报）**

**Return = 从当前时刻开始，到未来结束，所有奖励的**加权总和 。是智能体**真正想要最大化的目标**，不是单步奖励，而是**长期总收益**。

数学定义：$$G_t = r_t + \gamma\,r_{t+1} + \gamma^2\,r_{t+2} + \gamma^3\,r_{t+3} + \dots$$

- $$G_t$$：时刻$$t$$的**回报**

- $$r_t$$：即时奖励

- $$\boldsymbol{\gamma}$$：**折扣因子**（$$0 \le \gamma \le 1$$）

  - 越靠近 1：越重视长远

  - 越靠近 0：越短视，只看眼前

Trajectory 与 Return 的关系：给定一条轨迹$$\tau$$，就能**唯一算出**它的 Return：$$G_0 = \sum_{k=0}^\infty \gamma^k r_k$$

---

**Episode（回合）**

Episode = 一次完整的交互过程：从「开始」到「结束」

只要满足：

- 有**明确起点**
- 有**明确终点**（结束条件）
- 中间是一串连续的状态、动作、奖励

这一整段，就叫一个 **Episode**。

Episode与 Trajectory 的关系：**几乎可以认为：**1 Episode = 1 Trajectory

严格一点：

- **Episode** 强调**一局、一轮、一次试验**
- **Trajectory** 强调**这一局里的具体数据序列**

**两种强化学习任务分类**：

> 1. **Episodic Task（回合式任务）**
>
> 有开始、有结束，一局一局来。
>
> - 围棋、游戏、迷宫、CartPole
> - 我们目前学的几乎都是这种
>
> 2. **Continuing Task（连续任务）**
>
> **永远不结束**，没有 episode，一直跑。
>
> - 机器人持续走路、工厂控制、股票交易
> - 用 discount return Gt 来处理无限长序列

---

现在已经掌握**强化学习最底层全部基础概念**了：

1. **State $$s$$**：当前局面
2. **Action $$a$$**：做什么
3. **Policy $$\pi$$**：状态→动作的规则
4. **Transition $$P$$**：动作导致状态变化
5. **Reward $$r$$**：单步好坏
6. **Trajectory $$\tau$$**：一整段交互历史
7. **Return$$G$$**：长期总收益（优化目标）

完整流程：

$$\underbrace{s_t}_{\text{状态}}
\xrightarrow{\pi(a_t|s_t)}
\underbrace{a_t}_{\text{动作}}
\xrightarrow{P(s_{t+1}|s_t,a_t)}
\underbrace{s_{t+1},r_t}_{\text{新状态+奖励}}
\xrightarrow{\dots}
\underbrace{\tau}_{\text{轨迹}}
\xrightarrow{\sum\gamma^k r_k}
\underbrace{G_t}_{\text{回报}}$$

---



## MDP马尔可夫决策过程

**Markov Decision Process，马尔可夫决策过程，简称 MDP**。

MDP 是对**序贯决策问题**的数学描述：智能体在**具有马尔可夫性质**的环境中，通过**选动作**获得**奖励**，并**转移到下一状态**，目标是**最大化长期回报**。

### MDP 的 5 个核心组成（五元组）

一个 MDP 完全由这 5 个东西定义：$$\mathcal{M} = (\mathcal{S},\ \mathcal{A},\ P,\ R,\ \gamma)$$

1. $$\mathcal{S}$$：State Space 状态空间：所有可能状态的集合。

2. $$\mathcal{A}$$：Action Space 动作空间：所有可能动作的集合。

3. $$P$$：State Transition 状态转移概率：$$P(s' \mid s,\ a)$$。在状态 s 做动作 a，转移到 s′ 的概率。

4. $$R$$：Reward 奖励函数：$$R(s,\ a,\ s')$$。执行一步后得到的**即时奖励**。

5. $$\gamma$$：Discount factor 折扣因子：$$0 \le \gamma \le 1$$。对未来奖励的衰减权重，用于计算 Return。

### 马尔可夫性质

Markov Property = 未来只依赖当前，不依赖历史

$$P(s_{t+1} \mid s_t,a_t,s_{t-1},a_{t-1},\dots) = P(s_{t+1} \mid s_t,a_t)$$

**只要知道【现在状态 + 现在动作】，就足够预测未来，不需要过去的所有轨迹。**强化学习**全部建立在这个假设上**。

### MDP 的完整交互流程

这就是强化学习**每一步**在做的事：

$$s_t
\;\xrightarrow{a_t\sim\pi(a_t|s_t)}\;
(s_{t+1},\,r_t)
\;\sim\; P(s'|s_t,a_t),\; R(s_t,a_t)$$

一步步拆解：

1. 处于状态 $$s_t$$
2. 按策略 $$\pi$$选动作 $$a_t$$
3. 环境按转移概率$$P$$跳到$$s_{t+1}$$
4. 环境给出奖励 $$r_t$$
5. 进入下一时间步$$t+1$$
6. 直到 episode 结束

**MDP 的最终目标**：

给定 MDP：$$(\mathcal{S},\mathcal{A},P,R,\gamma)$$。求一个**最优策略 $$\pi^*$$**，使得**从任意状态出发的长期期望回报最大**：$$\pi^* = \arg\max_\pi \mathbb{E}_\pi\left[ G_0 \right]$$

所有强化学习算法，都只是在解 MDP 的不同方法。



---



# 第二章 贝尔曼公式

**Question：return有什么用？**：

> Reward 告诉你 “现在好不好”，Return 告诉你 “长远好不好”。
>
> 强化学习的目标，本质就是最大化 Return。



### 贝尔曼方程

贝尔曼方程 = **价值函数的递归分解式**：当前状态的价值 = 即时奖励 + 下一状态的折扣价值

它把  长期回报（Return）拆成：**眼前一步 + 未来剩下所有步**

这是**强化学习里唯一真正重要的等式**。

**价值函数**

>1. **状态价值函数$$V^\pi(s)$$**：$$V^\pi(s) = \mathbb{E}_\pi\!\left[\,G_t \,\big|\, s_t = s\,\right]$$
>
>   在状态$$s$$，按策略$$\pi$$走下去，**未来总回报的期望**。
>
>2. **动作价值函数 $$Q^\pi(s,a)$$**
>
>   在状态$$s$$**选动作 $$a$$**，再按策略 π 走下去，**未来总回报的期望**。
>
>贝尔曼方程就是**这两个函数的自递归公式**。
>
>价值函数本质就是对 Return 的期望估计。



**贝尔曼方程：**

>1. **状态价值函数的贝尔曼方程**
>
>$$V^\pi(s) = \mathbb{E}_\pi\big[\,r_t + \gamma\,V^\pi(s_{t+1})\,\big]$$
>
>**直观翻译：**这个状态有多好 = 这一步能拿到的奖励 * 下一个状态有多好（打个折 γ）
>
>2. **动作价值函数的贝尔曼方程**
>
>$$Q^\pi(s,a) = \mathbb{E}\big[\,r + \gamma\,V^\pi(s')\,\big]$$
>
>或者写成**自递归**（更重要）：
>
>$$Q^\pi(s,a) = \mathbb{E}\big[\,r + \gamma\,\max_{a'} Q^\pi(s',a')\,\big]$$
>
>这就是 **Q-learning 的源头**。

---

### State Value

**State Value（状态价值函数）$$\boldsymbol{V^\pi(s)}$$**= **从状态 s 出发，按照策略 π 一直走下去，未来能拿到的「期望长期回报 Return」**

通俗理解就是：这个状态 s 未来 “值多少钱”

数学定义：$$V^\pi(s) = \mathbb{E}_\pi\left[ G_t \mid s_t = s \right]$$  其中$$G_t = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \dots$$

- 和 Return 的关系：

  - **Return$$G_t$$**：**实际**拿到的长期总分（一条轨迹）

  - **State Value$$V(s)$$**：**平均 / 期望**的长期总分（很多条轨迹平均）

  因为环境 / 策略可能随机，所以用**期望**。

- StateValue有什么用？：

  > **判断状态好坏**（哪里前途好）
  >
  > **指导策略改进**：往 V 大的状态走
  >
  > **价值迭代、策略迭代的核心**

- 状态价值函数的贝尔曼方程
  $$V^\pi(s) = \mathbb{E}_\pi\big[\,r_t + \gamma\,V^\pi(s_{t+1})\,\big]$$

---

### 贝尔曼公式推导

首先基础公式：

> **折扣回报 Return**
>
> $$G_t = r_t + \gamma\,r_{t+1} + \gamma^2\,r_{t+2} + \dots$$
>
> **状态价值函数 State Value**
>
> $$V^\pi(s) = \mathbb{E}_\pi\!\left[\,G_t \,\big|\, s_t = s\,\right]$$
>
> **Return 的递推式**（关键拆分）
>
> $$G_t = r_t + \gamma\,G_{t+1}$$
>
> 把未来无穷多项，拆成**当下一步** + **剩下所有未来**。

正式推导：

> 由定义：
>
> $$V^\pi(s) = \mathbb{E}_\pi\!\left[\,G_t \,\big|\, s_t = s\,\right]$$
>
> 代入 **Return 递推式**$$G_t = r_t + \gamma G_{t+1}$$：
>
> $$V^\pi(s)
> = \mathbb{E}_\pi\!\left[\, r_t + \gamma\,G_{t+1} \,\big|\, s_t = s \,\right]$$
>
> 期望**线性展开**（期望的和 = 和的期望）：
>
> $$V^\pi(s)
> = \mathbb{E}_\pi\!\left[\,r_t \,\big|\, s_t=s\,\right]+\gamma\,\mathbb{E}_\pi\!\left[\,G_{t+1} \,\big|\, s_t=s\,\right]$$
>
> 看第二项：
>
> $$\mathbb{E}_\pi\!\left[\,G_{t+1} \,\big|\, s_t=s\,\right]
> = \mathbb{E}_\pi\!\left[\;
> \mathbb{E}_\pi\!\left[\,G_{t+1}\,\big|\,s_{t+1}\,\right]
> \;\big|\; s_t=s\;\right]$$
>
> （**全期望公式**：外层期望套内层期望）
>
> 而根据**价值函数定义**：
>
> $$\mathbb{E}_\pi\!\left[\,G_{t+1}\,\big|\,s_{t+1}=s'\,\right] = V^\pi(s')$$
>
> 所以第二项就变成：
>
> $$\mathbb{E}_\pi\!\left[\, V^\pi(s_{t+1}) \,\big|\, s_t=s \,\right]$$
>
> 最终合并得到贝尔曼方程：
>
> $$\boxed{
> V^\pi(s)
> = \mathbb{E}_\pi\!\left[\,
> r_t + \gamma\,V^\pi(s_{t+1})
> \,\big|\, s_t = s
> \,\right]
> }$$

在离散 MDP 里，把期望按动作、下一状态展开：

$$V^\pi(s) = \sum_a \pi(a|s)
\sum_{s'} P(s'|s,a)
\Big( R(s,a,s') + \gamma V^\pi(s') \Big)$$

---



### **贝尔曼一般公式（展开后的贝尔曼价值方程）：**

$$v_\pi(s) = \sum_{a} \pi(a|s) \left[ \sum_{r} p(r|s,a) r + \gamma \sum_{s'} p(s'|s,a) v_\pi(s') \right]$$

1.最外层：$$\displaystyle \sum_{a} \pi(a|s)$$

- $$\pi(a|s)$$：**策略（Policy）**。

含义：在状态 $s$ ，智能体选择动作 a 的概率。

- 这代表了智能体的**行为准则**。
- 因为策略通常是随机的（比如向左30%，向右70%），所以我们要对所有动作$$a$$求和。

- $$\displaystyle \sum_{a}$$：**全动作求和**。

含义：把在状态$$s$$下所有**合法动作**都考虑进去，计算它们的加权平均值。

2.中间括号：方括号内的部分

这部分描述了：**选了动作**$a$**之后，会发生什么？**

- **第一部分：**$$\displaystyle \sum_{r} p(r|s,a) $$

  - $$p(r|s,a)$$：**奖励转移概率**。
  -  含义：在状态 $s$做动作 后，获得奖励$r$的概率。
  - $r$：具体的奖励数值。
  - $$\displaystyle \sum_{r}$$：全奖励求和。
  - **直观理解**：这是**期望即时奖励**（Expected Immediate Reward）。意思是做了这个动作后，**平均能拿到多少分**。

- **第二部分：**$$\gamma \displaystyle \sum_{s'} p(s'|s,a) v_\pi(s')$$

  - $$p(s'|s,a)$$：**状态转移概率**。

    含义：在状态 s 做动作 a 后，环境跳转到下一状态 s′ 的概率。

  - $$v_\pi(s')$$：**下一状态的价值**。

     含义：跳到 s′ 后，从那里开始到结束还能值多少钱。。

  - $$\gamma$$：**折扣因子**（Discount Factor）。

     作用：把未来的价值“打折”，既保证数学收敛，也体现“眼前的奖励比未来更值钱”。

  - **直观理解**：这是**期望未来价值**。意思是做了这个动作后，**未来还有多少潜力**。



3.逻辑链条总结（数学直觉）

**策略层**：智能体先决定**选哪个动作**$$\sum_a \pi(a|s)$$。

**环境层**：动作选好了，环境开始运行。

- 环境给出**奖励**（第一部分：$$\sum_r p(r|s,a) r$$。
- 环境跳转到**下一状态**（第二部分：$$\sum_{s'} p(s'|s,a) v_\pi(s')$$。

**递归闭环**：把奖励和未来价值加起来，就是当前状态的价值。



