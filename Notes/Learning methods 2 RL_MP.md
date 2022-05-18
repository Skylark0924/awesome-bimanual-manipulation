
# Reinforcement Learning + Movement Primitives
1. Reinforcement Learning Bimanual Robot Skills
2. Bimanual robot skills: MP encoding, dimensionality reduction and reinforcement learning

上述两篇文章提出了以DMP和ProMP作为轨迹规划方法、Dual REPS作为强化学习控制方法、以EM为降维方法的双臂机器人控制框架。
![](https://pic4.zhimg.com/80/v2-9393208c2fc8b5a5129b657d12c64790.png)

# Reinforcement Learning Bimanual Robot Skills
![](https://pic4.zhimg.com/80/v2-ff05dee9526a8bb3e54b14832793cda6.png)

[Book Link](http://link.springer.com/10.1007/978-3-030-26326-3)

[[TOC]]

本书为强化学习与双臂机器人的专著，且是2020年的新作，保持了对研究热点追踪的实时性。本书主要内容可分为以下两部分：
1. 第一部分关注 kinematics, workspace manipulability maximization and control。其中最主要的是 compliant redundant robot control (兼容冗余机器人控制)，它基于著名的 Closed-Loop Inverse Kinematics (CLIK) algorithms 算法来求解逆运动学；
2. 第二部分关注 motion characterization and learning。其中主要是关于 reinforcement learning with movement primitives （具有运动原语的强化学习），它基于一种新的 Dual Relative Entropy Policy Search (DREPS) 算法，用于 robot motion 的快速学习，同时减少运动参数化的维度。

由此可以看出，本书是以自底向上的顺序介绍了双臂机器人控制系统相关的研究成果，尤其是展示了其对 CLIK 算法的修正（第一部分，CLIK 可能会产生 numerical instability 或 oscillations around a solution）以及强化学习与运动基元的结合（第二部分，同时展示了降维能力）。

### Introduction
二十世纪中叶，神经生理学家 Nikolai A. Bernstein 研究体力劳动中的人体运动，旨在通过研究人类运动来帮助提高生产力，并指出
> *We humans activate in a coordinated manner those muscles that we cannot control individually*

人体中约有 640 块肌肉，这是人类大脑无法独立控制的自由度数。肌肉通过韧带和肌腱控制人体骨骼的各种运动。因此，人类学习与任务相关的协同模式。然后将这些模式存储在人脑中并在需要时执行，并在每次执行后重新评估以进一步改进。

> 以乒乓球运动员训练过程为例：
> 1. 模仿别人的挥杆；
> 2. 每次执行挥杆动作都有可能进行探索和创新；
> 3. 评估每次执行的结果
> 4. 如果这种结果积极，那么储存在人类大脑中的肌肉协同作用就会被改变。否则，就不会改变。

与此同时，1920年代，作家 Karel Capek 给出了机器人的定义。随后在1960年代，学术界逐渐对赋予机器人更高的智力产生兴趣。到了二十世纪的最后二十年，强化学习成为研究机器人智能的主要技术手段。然而，现在通过求解冗余机械手（如拟人手臂）的逆运动学来找到将机器人末端执行器置于特定姿势的合适关节位置仍是一个复杂的问题。一旦解决了运动学问题，就需要考虑控制问题，特别是在未建模场景的情况下，包括与人类和/或可变形物体的交互。希望机器人具有所谓的**柔顺控制器**，即：一种能够在施加最小扭矩的同时精确跟踪给定命令的控制方案，以免沿途伤害任何物体/人。学习机器人技能是一个难题，可以通过多种方式解决。
最常见的方法是从演示中学习 (LfD)，其中向机器人展示解决任务的初始方式，然后尝试复制、改进和/或使其适应可变条件。

![](https://pic4.zhimg.com/80/v2-8170c6b79fc68877c80b688261c1a4c6.png)

> 又是一章精彩的开篇，


## 6 Sampling Efficiency in Learning Robot Motion

REPS 使用KL散度来限制新旧policy之间的差异。然而，KL参数的顺序 具有平均解之间的行为，有时会由于两个或多个接近的局部最优之间的竞争而找到非最优解。这种 KL 优化生成的解决方案产生高概率，其中数据呈现高回报，因此在高斯分布中的模式之间进行平均。使用反向排序将有助于更快地找到单个解决方案，因为这种方法将找到具有低概率值的解决方案，其中数据具有低奖励 [10]，因此仅关注一种模式。然而，没有使用 KL(q‖π) 代替 KL(π‖q) 的封闭 REPS 解决方案。虽然 KL 的后一种用法似乎更适合大多数 RL 应用程序，但其解析不可解性使其应用程序不切实际。
在本章中，我们将展示我们提出的 DREPS 算法避免了上述解决方案之间的平均行为，表现出良好的能力，可以避免陷入局部最大值之间。

为了让我们提出的 DREPS 算法适应 RL 的当前趋势，我们假设策略被编码为多元正态随机分布。
这种编码，以及用高斯分布表示聚类样本，使我们能够解析地求解所得方程并获得封闭形式的解。

### The Dual REPS Algorithm
双REPS（DREPS）的想法是**使用集群糟糕表现不佳的数据样本作为一个令人厌恶的领域（a repulsive field），成效良好者作为策略的搜索算法中的吸引子（attractor）。**

为实现上述目标，假设可以从一组实验中聚类数据，并将这些信息编码为优化问题中的约束。添加这个新约束的自然方法是在坏/好数据集群和新策略之间设置最小/最大 KL 散度。这导致了二分效应，在给定可用样本的情况下，为策略更新寻找最佳解决方案时限制了搜索空间。下式即为添加了约束项的策略更新公式：
![](https://pic4.zhimg.com/80/v2-73ab6d66cccb838ae76819906678562d.png)
1. 第一个限制（排斥）防止新策略停留在优化空间的次优区域，而不是通过赋予它们零重要性来忽略低性能样本（如 REPS 中发生的那样）。
2. 当存在多个最优值时，第二个限制允许策略搜索的贪婪行为，有助于在可能的情况下更快地收敛


## 7 Dimensionality Reduction with Movement Primitives
将RL方法与MPs结合时，可以考虑通过以下角度对参数进行降维：
1. Model availability 然而，在非刚性对象的操纵的情况下，或更一般地，当精确的模型是不可用的，减少的参数和的推出的数量是重要的。
2. Exploration Constraints  某些探索值可能会导致真实机器人的危险运动，例如强烈的振荡和突然的加速度变化。此外，某些任务可能不依赖于机器人的所有自由度 (DoF)，这意味着使用的 RL 算法可能正在探索与任务无关的运动。
3. Parameter dimensionality  更多的参数通常可以更好地拟合初始运动特征，但是在如此高维空间中进行学习探索会导致学习速度变慢。

本章将介绍如何在ProMPs和DMPs上分别应用降维方法。

### Dimensionality Reduction for ProMPs
我们使用概率降维技术，使用期望最大化 (EM) 从一组给定的演示中提取未知的协同作用，同时最大化关于此类演示运动或奖励加权机器人探索性执行的对数似然函数。目前已经有了一些工作使用 PCA 和 PPCA 作为降维方式，我们直接用 EM 提取 latent space，不需要 PCA。 PCA 仅用作 EM 方法的初始化。


