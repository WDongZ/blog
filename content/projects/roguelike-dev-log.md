---
title: "技术复盘：从零构建以撒式 2D Roguelike 框架"
date: 2026-01-12
description: "基于 Unity 实现的类以撒随机地牢与道具系统开发实录"
# 下面是 PaperMod 主题的标准封面写法
cover:
    image: "/image/boss.png"  # 注意：这里路径是从 static 文件夹开始算的，见下文说明
    alt: "Boss 战演示"
    caption: "Boss AI 行为树与阶段转换演示"
---

# 技术复盘：从零构建以撒式 2D Roguelike 框架

## 🚀 项目概览

本项目是一款 2D 俯视角 Roguelike 动作游戏。开发核心目标是复刻类似《以撒的结合》的**程序化关卡生成**与**高度可堆叠的道具系统**。

- **引擎版本：** Unity 2022.3 LTS
- **核心挑战：** 随机地图的拓扑一致性、基于 ScriptableObject 的解耦道具系统、多层级 AI 行为树。

------

## 🛠 关键技术实现

### 1. 程序化随机房间生成算法 (Procedural Room Generation)

为了实现“以撒式”的布局，我放弃了简单的网格填充，采用了一种基于**深度优先搜索 (DFS) 变体**的增长算法。

- **逻辑流程：**
  1. 初始化中心起始房间。
  2. 利用队列维护待生成的出口，随机尝试向四周伸展。
  3. **约束控制：** 通过设置 `MaxRooms` 和 `DeadEnd` 概率，确保地图既不会无限延伸，也不会过于拥挤。
  4. **特殊房间放置：** 算法会在距离起点最远的分支末端自动计算并放置 **Boss 房**，在侧支放置 **商店房**。

------

### 2. 高扩展性道具与子弹修改器系统

Roguelike 的灵魂在于道具堆叠。为了避免 `if-else` 爆炸，我采用了**装饰器模式 (Decorator Pattern)** 与 **ScriptableObject** 相结合的方案。

- **技术亮点：**
  - **Data-Driven Design:** 每一个道具都是一个 ScriptableObject，包含修改属性（攻击力、攻速）和子弹特效（分裂、追踪、反弹）。
  - **计算管道：** 玩家射击时，子弹会经过一个 `ModifierChain`，依次应用当前所有道具的效果。
  - **解耦：** 道具系统与玩家逻辑完全解耦，新增一个道具只需新建一个配置文件，无需修改一行代码。

![alt text](/image/bullet.png)

------

### 3. 多样化 AI 决策逻辑：状态机与预警系统

针对普通小怪和复杂的 Boss，我实现了分层式的 AI 架构：

- **小怪 AI：** 基于有限状态机 (FSM)，处理“巡逻-追踪-攻击-逃跑”的转换。
- **Boss AI：** 引入了**权重随机行为机制**。Boss 会根据当前血量百分比切换阶段（Phase），并根据玩家距离权重触发不同的技能组。
- **视觉反馈：** 实现了集中目标相机抖动，提升了游戏的交互打击感。

![alt text](/image/boss.png)

------

### 4. 2D 光影渲染与小地图性能优化

- **光影氛围：** 利用 Unity URP 的 **2D Lights** 实现动态阴影，通过法线贴图增强环境质感。
- **小地图 (Minimap)：** 使用了独立的相机渲染层级（Layer Mask）结合 **Render Texture**。
- **性能优化：** 针对大量子弹同屏的情况，引入了简单的**对象池 (Object Pooling)**，有效避免了频繁 Instantiate/Destroy 带来的内存抖动。

![alt text](/image/map.png)
------

## 🧠 难点攻克：如何解决房间重叠？

在随机生成逻辑开发初期，房间经常重叠。 **解决方案：** 我引入了一个 2D 坐标哈希表 `Dictionary<Vector2Int, Room>`。在生成新房间前，先进行 `ContainsKey` 检查。同时利用**位掩码 (Bitmasking)** 技术自动判断房间四个方向的墙壁连接状态，实现了无缝的自动门系统。

------

## 📈 总结与展望

通过本项目，我深入实践了数据驱动设计、复杂算法在游戏环境下的落地以及性能调优技巧。

**未来迭代方向：**

1. 引入 **A\* 寻路** 替代当前的简单线性追踪。
2. 实验使用 **Behavior Tree (行为树)** 插件重构 Boss AI 以获得更高复杂度的行为。

------

### 🔗 源码与文档

- **GitHub Repo:** [点击跳转到项目仓库](https://github.com/WDongZ/2D-Game-Development-NUS/)
- **演示视频:** [Bilibili 链接](https://www.bilibili.com/video/BV13W42197Ta/?spm_id_from=333.1387.homepage.video_card.click)
