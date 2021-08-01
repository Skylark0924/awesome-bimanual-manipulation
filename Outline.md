# A comprehensive survey on bimanual robotic manipulation

[TOC]



## Dual arm manipulation—A survey

### Introduction

The added complexity of dual or multi-arm manipulation presents many challenges that may not be present in the single manipulator case. This higher complexity means that dual arm manipulation requires **more advanced system integration, high level planning and reasoning, as well as viable control approaches**. 

![image-20210801131958957](Outline.assets/image-20210801131958957.png)

#### Advantages of dual-arm robots

1. Similarity to operator
2. Flexibility and stiffness
3. Manipulability
4. Cognitive motivation 探索类人物理交互与认知的关系
5. Human form factor  据称类人型台的机器人性能更好

#### Category

- non-coordinated manipulation  双臂执行不同任务
- coordinated manipulation  双臂执行同一任务的不同部分
  - goal-coordinated manipulation  手臂之间没有物理交互，类似双手打字
  - bimanual manipulation  存在物理交互 （图1）

![image-20210801134048628](Outline.assets/image-20210801134048628.png)

第一大类与单臂无异；第二大类才是本文主题

#### Applications

**Domestic**

1. folding laundry
2. providing attendant care and kitchen support for the elderly
3. performs cooperative manipulation task with a human, or transports the object autonomously

**Industrial**

1. parts assembly
2. crew-assisting robotic system for space exploration

#### Hardware

![image-20210801140309492](Outline.assets/image-20210801140309492.png)

### Methodology

#### Modelling

根据多臂末端与物体的交互方式，可以分为两种协作操作任务：

1. Cooperative robot manipulators holding an object with fixed grasp points (Fig. 2(a)) 即机械臂与物体无相对运动
2. Cooperative robot manipulators holding an object by means of contact points or contact areas (Fig. 2(b))  即机械臂与物体可以相对运动

![](Outline.assets/image-20210801154307341.png)

Grasp Matrix 描述了接触点或抓取点处的速度和力与映射到物体质心的速度和力之间的动静力学（kineto-static）关系。以图2(a)为例，

![image-20210801231256090](Outline.assets/image-20210801231256090.png)



接触建模确定了末端执行器扳手在接触点处将到达物体的分量，从而确定了传递到物体的速度分量。有一下三种主要类型的接触模型：

1. Frictionless point contact
2. Frictional point contact or hard finger model
3. Soft contact or soft finger model

接触方式对第二种协作操作任务，即机械臂与物体存在相对运动，在建模时候的影响更大。上述抓取形式及接触类型（在非固定抓取的情况下）用于导出控制操作任务的约束。

#### Control

**trajectory tracking**

1. input–output linearization
2. Hybrid force/position control
3. impedance control
4. sliding mode control, robust adaptive control
   as well as neuro-adaptive control
5. Intelligent control using neural networks and fuzzy systems

**regulation**

**visual servoing**

视觉伺服在控制回路中的参与方式可以分为

1. image-based  直接使用观测与理想位置的2D图像特征之间的差异作为控制反馈
2. position-based  使用物体被观测到的3D位姿与理想位姿之间的差异作为控制反馈（该位姿从RGB/D图像估计得到）
3. hybrid methods

此外，相机的安装位置也有几种：

1. End-effector mounted  装在机械臂末端
2. Fixed in the workspace  固定在工作空间的某个位置
3. Active head  装在机器人头上（常见于家庭中的拟人机器人）

![相机安装位置](Outline.assets/image-20210801180507774.png)

#### Planning

motion, trajectory and manipulation planning

1. Randomized potential field planning techniques with the introduction of transit (where the arms move independently and the object remains static) and transfer (where the arms and the object form a closed kinematic chain) subpaths
2. Randomized Path Planner algorithm
3. RRT

#### Learning

1. programming by demonstration (PbD) 
2. Hidden Markov Models and Gaussian
   Mixture Regression
3. combines Dynamical Systems movement control with a Programming by Demonstration approach
4. learning a policy which maps the state to a control vector

### Future direction

1. integration of elements from systems theory with tools from **cognitive** methodologies
2. Is it meaningful to mimic the (proven successful) configuration of biological systems, such as humans? Should more advanced sensors be used to cover deficiencies in planning, modelling, and control, or should the latter be developed to cover deficiencies in sensors?
3. **high- level reasoning / scene understanding**