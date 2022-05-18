#! https://zhuanlan.zhihu.com/p/400106188
# 双臂机器人的学习方法Ⅰ：模仿学习

> Bimanual robot learning methodsⅠ：Imitation learning

推荐两篇近期关于双臂机器人模仿学习的 papers，目录如下（什么时候专栏文章才能拥有目录！！）

![](https://pic4.zhimg.com/80/v2-c829842bf5d54a702eaba4a7ab2e0629.png)

## Deep Imitation Learning for Bimanual Robotic Manipulation
![](https://pic4.zhimg.com/80/v2-0f1bf09b8110e04b6c9285d9f401d482.png)

[Paper Link](http://arxiv.org/abs/2010.05134)

### Brief introduction
本文是针对双臂机器人的深度模仿学习框架，旨在提高机器人操作与示范数据不同位置的物体时的泛化性。本文提出借助从复杂动态的环境（即任务需要多个物体交互才能完成目标）中捕获关系信息（即轨迹涉及与环境中其他物体的关系）的方式。具体思路如下：

1. 将运动过程分解为多个基本的运动基元；
2. 使用循环图神经网络对每个基元进行参数化以捕获捕获机器人-机器人和机器人-物体的交互；
3. 机器人的控制方法被分为按顺序组合基元的 high-level planner 和结合基元动力学和反向运动学控制的 low-level controller。

![](https://pic4.zhimg.com/80/v2-35a1d63c84ba40af69345013bf305477.png)

### Background
Imitation learning 是根据专家 policy $\pi_E$ 得到的一组示范数据，学习一个可以模仿该专家策略的$\pi_\theta$。对于 long horizon problems，模仿学习需要大量的示范数据，这可以通过 hierarchical imitation learning （分层模仿学习）来缓解。其将策略分为: a high-level planning policy and a low-level control policy 两层。High-level 策略 $\pi_h$ 用于生成基元序列。每一个基元策略 $\pi_{p^k}$ 可以代表机器人一项技能，可以借此生成该技能下机器人的 low-level 轨迹。

### Methodology
本文就是遵循 HIL 的精神，提出了一个分层的双臂模仿学习框架。该框架将要预测的 $N$ 个状态，均分为 $K$ 个基元，每个基元的轨迹包含 $M=N/K$ 个状态。A high-level planning
 根据先前的状态 $(s_0, ...,s_{t−1}) \mapsto p_t$ 选择基元。
A low-level primitive dynamics model 使用选定的基元 $p_t=p^k$，预测接下来的 $M$ 步 $s_{t−1} \mapsto (s^k_t ...s^k_{t+M})$。

![](https://pic4.zhimg.com/80/v2-22c216b92fed335e21d25673189fd48e.png)

本文的 low-level 和 high-level 模型差不多，所以就以介绍 low-level 为主。

#### Low-level Control Model

本文通过 variational auto-encoder 引入了一个随机分量来生成隐状态分布 $Z_t \sim \mathcal{N}(\mu_{z_t},\sigma_{z_t})$。Decoder 从分布中采样隐状态 $z_t$ 并生成输出序列。以此为基础，在多个方面进行创新，以执行高精度的双手操作任务。

**Relational Features (Int)**

Vanilla RNN encoder-decoder 模型假设不同的输入特征是独立的，然而机械臂和物体之间具有很强的依赖性。为了捕捉这种关系依赖，本文在 decoder 中引入了一个图注意力（GAT）层，从而形成 graph RNN（注：这和GraphRNN不是一个东西）。该模块允许我们学习捕获物体之间的关系信息。

**Residual Connection (Res)**

该模型的另一个关键组件是 residual skip connection，它将目标物体的特征（例如要移动的桌子）连接到 encoder GRU 的最后一个隐藏层，有助于强调任务目标，或者说目标物体特征。

**Modular Movement Primitives (Multi)**

基元代表基本的运动模式，例如抓握、举起和转动。本文每个基元设计了一个单独的神经网络模块，而非一个统一的模型。因此，每个神经网络（结构相同）都会捕获特定类型的动态基元。
每个基元的最后一个状态用作下一个基元的初始状态，这是由 high-level planning 预测的。

#### High-level Planning Model

与 low-level controller 类似，planner 使用相同的 graph RNN 模型，并在输入中引入关系特征 Int。但是 planner 省略了残差连接，因为本文发现这对基元识别并非必要。

![](https://pic4.zhimg.com/80/v2-c54ef2272e615da5bd7da100c464b8ff.png)

#### Graph
这里详细讲一下本文是如何将环境和机器人表征为 graph 的。

节点对应于状态中的一个物体特征，即 x、y、z 坐标和四个四元数。

![](https://pic4.zhimg.com/80/v2-2e9a01d7d84095dd60f1d26d87b646b7.png)

所以，上图机器人搬桌子的任务中，左手、右手以及桌子分别以7个节点代表，通过 GAT 的 attention weight 学习，得到了边上不同的权重，只有边上权重大于 8% 的边才会被画出来。


### Training
以上两个层级均通过监督学习进行训练。其中，high-level 根据手动标注的示范数据，学习序列状态与不同基元类别之间的关系。

### Experiments
本文使用 PyBullet 仿真器以及 Baxter 机器人进行仿真试验。

示范数据在模拟器中被手动标记为多个基元，每个基元函数都被参数化，因此可以适应不同的起点和重点坐标。选择基元，使它们每个都具有不同的轨迹动态（即向上提升与侧向移动）。每个基元为机器人抓手生成了 10-12 个状态的序列。状态的数量是通过模拟实验选择出来的，每个基元需要最少的状态来产生平滑的非线性轨迹，而过多的状态会增加预测的复杂性。得到预测的状态序列之后，即可使用逆运动学将状态转换为机器人模拟器的命令动作。本文给出了两个模拟任务，如下：

**Table Lifting Task**

![](https://pic4.zhimg.com/80/v2-c4463a951d69f21c7afb94a6dd4f5e18.png)

**Peg-in-Hole Task**

![](https://pic4.zhimg.com/80/v2-fcc12676119e4d90970f0f12e66d16aa.png)


### Conclusion
本文结合了图神经网络对环境关系的提取能力，以及基元对单一技能的刻画能力，使机器人操作的整个流程变得更加可解释了一些。本文还提出，可以在从视觉输入中估计物体姿态，以及自动判定基元这两个方向上对机器人模仿学习进行改进。

___


## Transformer-based deep imitation learning for dual-arm robot manipulation
![](https://pic4.zhimg.com/80/v2-c9b19e719fe3c742f32e3cfa482bc0f5.png)

[Paper Link](http://arxiv.org/abs/2108.00385)


### Brief Introduction
本文是从减轻传感器输入对运动学状态的干扰、压缩 image-based task 状态维度的角度出发，利用 transformer 计算按顺序输入的元素之间的依赖性，并使其专注于重要元素。

在这个框架中，each element of the sensory inputs (gaze position, left arm state, and right arm state) 以及 foveated image embedding 被输入到 Transformer 以确定应该注意哪个元素。这种注意力使输出策略对传感器输入中不必要的干扰具有鲁棒性。

本文分别在 an **uncoordinated** manipulation task (two arms executing different tasks), a **goal-coordinated** manipulation task (both arms solving the same task but not physically interacting with each other), and **bimanual** tasks (both arms physically interacting to solve the task) 对所提出的基于 Transformer 的深度模仿学习架构分别进行了验证。结果表明，Transformer-based self-attention mechanism 可以改善双臂操作能力。

![](https://pic4.zhimg.com/80/v2-133da1970f9c5385ca96c3007f2e170c.png)

### Methodology
文中提到，本文是第一个在真实机器人环境上尝试 self-attention-based DIL 方法的论文。所以，先看看**硬件配置**：

1. **两个 UR5 机械臂**（有人类遥操作）
2. 一个**头戴式显示器** (HMD) 提供从安装在机器人上的立体相机捕获的视觉。
3. 在遥操作期间，人类的凝视区域 (the human gaze) 由安装在 HMD 中的**眼动仪**测量
4. 左相机图像被调整为 256 × 256（称为全局图像）并以 10 Hz 的频率记录左眼的二维注视坐标和双臂的机器人运动学状态
5. 每个机器人运动学状态被定义为一个十维向量，末端执行器位置（三维）、方向（六维，使用三维欧拉角的余弦和正弦组合表示，以防止状态发生剧烈变化时角度超过 2π），以及每个手臂的夹爪角度（一维）

上面第3条出现了一个高端名词——**眼动仪**，这将有效地屏蔽掉大部分视觉干扰，所以接下来就是如何在机器人自主任务中预测注视位置。

#### Gaze position prediction
本文引用了这篇 paper 对 human gaze 的处理，即使用 mixture density network (MDN) 来估计注视位置的概率分布），这是一种将高斯混合模型 (GMM) 拟合到目标中的神经网络结构。

![](https://pic4.zhimg.com/80/v2-4609ae829aba92fa77c9da7a3e766e79.png)

以上神经网络通过示范数据中的人类 gaze 数据进行监督训练。在 test 过程中，从平均位置（μ）中选择单个注视位置，推断出 GMM 的概率最高：

$$
\text { gaze }=\mu^{\arg \max \left(p^{i}\right)}
$$

#### Transformer-based dual-arm imitation learning
所提出的框架如图1所示。

1. gaze predictor 预测整个 256 × 256 图像中的二维凝视位置，并据此裁剪成 64 x 64；
2. 由此产生的特征通过全局平均池化层（MAP），该层将每个特征平均为一个值，以减少参数数量；
3. 预测的凝视位置和左右机器人操纵状态被串联成 22 维状态；
4. 每个 1+22 维状态元素与代表位置的 one hot 向量通过线性投影，生成具有所学位置嵌入的 64 维状表征；
5. 图像嵌入和状态表征 concat 在一起,并使用三层 Transformer encoder 进行编码；
6. encoded feature 被展平，并通过一个 MLP 来预测两臂的操作。

### Experiments
本文设计了 Pick, BoxPush, ChangeHands, KnotTying 四种任务。其中，Pick 是 uncoordinated 任务、KnotTying 是 goal-coordinated 任务，BoxPush 和 ChangeHands 是bimanual 任务。

![](https://pic4.zhimg.com/80/v2-cbaf21d6c09386958cdd768494e2d835.png)

### Conclusion
本文提出了基于 Transformer 的自注意力机制，用于对真正的机器人双臂操作任务进行深度模仿学习。由于自注意力机制滤掉了与当前任务无关的传感器输入，因此可以抑制传感器输入的干扰。同时，本文提到该方法还可以拓展到多臂机器人或类人机器人上。文中还提到，未来可以考虑引入力反馈来丰富该控制框架。