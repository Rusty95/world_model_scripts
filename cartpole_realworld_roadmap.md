# CartPole 实体项目路线建议

## 1. 项目定位

本项目的核心目标不是单纯做一个能平衡的倒立摆，而是搭建一个可以验证以下流程的真实实验平台：

```text
IsaacLab 仿真环境
    ↓
图像 / action 数据采集
    ↓
LeWorldModel 训练
    ↓
真实 CartPole 数据采集
    ↓
sim-to-real / real-world finetune
    ↓
world model prediction / planning
    ↓
真实系统部署验证
```

因此，当前阶段的目标应该收敛为：

> 先做一个能稳定采集真实视觉、action、state 数据的 CartPole 实验平台。

不要一开始就追求“LeWorldModel 直接控制真实 CartPole 成功平衡”。那个目标太大，容易把硬件、控制、模型、部署、安全等问题全部混在一起，导致很难 debug。

---

## 2. 当前已有基础

目前已经完成或基本打通的部分：

```text
强化学习数学原理入门
PPO / A2C / A3C 等算法理解
LeWorldModel 学习
IsaacLab 中搭建 CartPole 模型
采集仿真图像和 action
用于 LeWorldModel 训练的数据流程初步打通
```

这些已经覆盖了算法和仿真侧的主链路。

下一阶段应从“学习算法”切换到“构建实验系统”。

---

## 3. 当前迷茫的本质

现在会迷茫，是因为问题从单点算法变成了完整系统：

```text
RL 算法
World Model
IsaacLab
图像数据采集
真实硬件
电机控制
传感器同步
sim-to-real
模型部署
安全保护
实验记录
```

这些方向都重要，但当前不能同时展开。

现在最重要的是收缩问题，只保留一条主线：

> 真实 CartPole 数据采集平台。

只要这个平台成了，后面的 LeWorldModel 真实预测、sim-to-real、MPC、RL fine-tune、image-only control 都可以自然展开。

---

## 4. 当前阶段不要做什么

### 4.1 不要一开始追求真实平衡成功

第一版实体平台的验收标准不应该是：

```text
能不能倒立摆平衡
```

而应该是：

```text
能不能稳定采集真实 episode 数据
```

也就是：

```text
image_t
对应 action_t
对应 encoder state_t
对应 timestamp_t
```

能稳定保存、回放、训练。

---

### 4.2 不要一开始做 image-only

最终可以做 image-only policy，但第一版工程上必须保存 encoder 真值。

推荐设计：

```text
图像：给 LeWorldModel / policy 使用
encoder state：给 debug / baseline / 安全保护 / 评估使用
```

原因是，如果模型失败，需要区分问题来自哪里：

```text
图像采集失败？
视觉表征失败？
world model 预测失败？
policy / planner 失败？
action scale 错误？
电机响应不一致？
延迟太大？
```

没有 encoder state，debug 会非常困难。

---

### 4.3 不要一开始实体强化学习

真实系统上直接跑 RL 成本高、风险高、调试困难。

第一阶段应该使用：

```text
random action
scripted action
PID
LQR
manual disturbance
```

来采集真实数据。

等 world model 对真实图像有一定预测能力之后，再考虑真实闭环控制。

---

### 4.4 不要一开始把硬件做得过度复杂

第一版硬件不需要工业级精度。

优先级应该是：

```text
可运动
可观测
可记录
可复现
可急停
```

而不是：

```text
结构极致精密
控制性能极致高
材料极致昂贵
```

---

## 5. 项目主线拆分

建议把项目拆成三条线。

---

## 5.1 硬件线

目标：构建一个真实 CartPole 数据采集平台。

第一版硬件需要满足：

```text
小车能左右移动
摆杆能自由转动
能读取 cart position
能读取 pole angle
相机能拍到完整运动区域
能记录 action、image、encoder、timestamp
有物理急停
有两端限位保护
```

第一版验收标准：

```text
能稳定采集真实 episode 数据
```

而不是：

```text
能稳定平衡倒立摆
```

---

## 5.2 数据线

目标：统一仿真数据和真实数据格式。

IsaacLab 数据和真实硬件数据应该尽量使用同一套 dataloader。

推荐最小数据字段：

```text
obs_image_t
next_obs_image_t
action_t
done_t
timestamp_t
```

真实平台额外保存：

```text
cart_position_t
cart_velocity_t
pole_angle_t
pole_angular_velocity_t
motor_command_t
failure_reason_t
```

注意：encoder state 不一定直接给模型训练，但一定要保存。

它们用于：

```text
debug
性能评估
baseline controller
真实状态可视化
sim-to-real 参数校准
安全保护
```

---

## 5.3 模型线

目标：验证 LeWorldModel 在真实 CartPole 上的预测与部署价值。

建议按以下三个实验推进：

### 实验 A：sim-only prediction

```text
sim image_t + action_t -> sim image_t+1
```

目的：确认仿真数据训练流程没问题。

---

### 实验 B：real-only prediction

```text
real image_t + action_t -> real image_t+1
```

目的：确认模型能学习真实系统动力学和真实图像分布。

---

### 实验 C：sim pretrain + real finetune

```text
sim pretrain -> real finetune
```

目的：验证仿真预训练是否能提升真实数据效率。

这是该项目很有价值的一组实验。

---

## 6. 推荐项目阶段

## Stage 0：已有基础阶段

已完成内容：

```text
RL 基础
PPO / A2C / A3C
LeWorldModel 学习
IsaacLab CartPole
仿真 image/action 数据采集
训练流程初步打通
```

当前状态：可以进入真实系统阶段。

---

## Stage 1：真实数据采集平台

目标：让真实 CartPole 成为一个可采集数据的环境。

输出物：

```text
真实 CartPole 硬件平台
MCU 控制程序
相机采集程序
encoder 读取程序
episode 保存格式
episode replay 工具
```

验收标准：

```text
1. 小车可以左右移动
2. 摆杆可以自由转动
3. 能读取 cart position
4. 能读取 pole angle
5. 相机画面稳定
6. 每一帧图像都有对应 action 和 timestamp
7. 数据可以保存成 episode
8. 可以用 replay 脚本播放 episode
```

---

## Stage 2：真实世界模型

目标：让 LeWorldModel 能预测真实 CartPole 图像。

输出物：

```text
real-only world model
sim-only world model
sim-pretrain-real-finetune world model
one-step prediction 可视化
multi-step rollout 可视化
真实数据和仿真数据误差对比
```

核心指标：

```text
one-step prediction error
multi-step rollout error
latent consistency
真实图像预测质量
action-conditioned dynamics 是否正确
```

---

## Stage 3：模型辅助控制

目标：先用 world model 做 planning，而不是直接实体 RL。

推荐方式：MPC / shooting-based planning。

流程：

```text
当前图像 obs_t
    ↓
采样多组 action sequence
    ↓
用 world model rollout 预测未来
    ↓
根据 cost 选择最优 action sequence
    ↓
执行第一个 action
    ↓
下一帧重新规划
```

cost 可以先设计成：

```text
pole angle 越接近竖直越好
cart position 越接近中心越好
action 不要过大
不要接近轨道边界
```

第一版可以用 encoder state 计算 cost。

后续再尝试完全 image-based cost。

---

## Stage 4：真实闭环部署

目标：让模型控制真实系统，并具备安全保护。

部署链路建议：

```text
Camera
    ↓
Image preprocessing
    ↓
LeWorldModel / policy / planner
    ↓
Safety supervisor
    ↓
Action clipping
    ↓
MCU low-level controller
    ↓
Motor driver
    ↓
CartPole hardware
```

必须加入的安全机制：

```text
cart position limit
pole angle limit
action saturation limit
inference timeout detection
camera dropout detection
encoder dropout detection
emergency stop
baseline fallback controller
```

---

## 7. 第一版硬件建议

第一版硬件不要过度复杂。

建议方案：

```text
MGN12 / MGN15 直线导轨
GT2 同步带
闭环步进电机或 DC/BLDC servo
STM32 / Teensy / ESP32 控制板
摆杆角度编码器
小车位置编码器或电机编码器
USB 相机
24V 电源
限位开关
急停按钮
LED 补光
```

### 7.1 更稳妥的研究版

```text
线性导轨 + 同步带 + DC/BLDC servo + 双 encoder + 外置相机
```

优点：

```text
控制性能更好
更接近连续力控
更适合做 sim-to-real
更适合后续高质量实验
```

缺点：

```text
成本较高
驱动和调参更复杂
```

---

### 7.2 快速打通版

```text
线性导轨 + 同步带 + 闭环步进电机 + 摆杆 encoder + USB 相机
```

优点：

```text
实现快
成本低
调试简单
适合第一版
```

缺点：

```text
action 更像速度 / 位置控制
不完全等价于仿真里的 force action
高速动态性能有限
```

当前建议：

> 先用快速打通版做 M1，后续再升级执行器。

---

## 8. 第一版 BOM

### 8.1 机械结构

| 模块 | 物料 | 建议规格 | 数量 | 备注 |
|---|---|---:|---:|---|
| 主框架 | 2020 / 2040 铝型材 | 800 mm - 1200 mm | 2-4 | 轨道越长越好 |
| 线性导轨 | MGN12 / MGN15 | 800 mm - 1000 mm | 1-2 | MGN15 更稳 |
| 滑块 | MGN12H / MGN15H | 与导轨匹配 | 1-2 | 小车安装在滑块上 |
| 小车底板 | 铝板 / 碳板 / 3D 打印件 | 约 100×80 mm | 1 | 轻且刚性好 |
| 同步带 | GT2 / HTD-3M | 开环或闭环 | 1 | GT2 够第一版使用 |
| 同步轮 | 20T / 30T | 与电机轴匹配 | 2 | 一端主动，一端惰轮 |
| 张紧结构 | 可调电机座 / 滑槽 | 自制 | 1 | 皮带张紧很关键 |
| 摆杆 | 碳纤维管 / 铝管 | 300 mm - 500 mm | 1-3 | 建议准备多种长度 |
| 摆杆转轴 | 光轴 / 不锈钢轴 | 4 mm - 8 mm | 1 | 要低摩擦 |
| 摆杆轴承 | 624ZZ / 625ZZ / 608ZZ | 与轴匹配 | 2 | 减小 pivot friction |
| 限位缓冲 | 橡胶块 | 两端安装 | 2-4 | 防撞 |
| 相机支架 | 铝型材 / 三脚架 / 3D 打印件 | 固定外参 | 1 | 相机必须固定 |

---

### 8.2 执行器与电源

| 模块 | 物料 | 建议规格 | 数量 | 备注 |
|---|---|---:|---:|---|
| 电机 | 闭环步进 / DC servo / BLDC servo | 24V | 1 | 第一版闭环步进较简单 |
| 电机驱动器 | 闭环步进驱动 / ODrive / VESC / DC driver | 与电机匹配 | 1 | 根据电机选择 |
| 电源 | DC 开关电源 | 24V 5A - 15A | 1 | 电机功率决定电流 |
| 降压模块 | DC-DC buck | 24V → 5V / 12V | 1-2 | 给 MCU / 传感器供电 |
| 急停按钮 | 蘑菇头急停 | 常闭型优先 | 1 | 建议直接切电机电源 |
| 保险丝 | 直流保险丝 | 5A - 15A | 1 | 必须加 |
| 电源开关 | 带灯开关 | 24V | 1 | 方便实验 |

---

### 8.3 传感器

| 模块 | 物料 | 建议规格 | 数量 | 备注 |
|---|---|---:|---:|---|
| 摆杆角度编码器 | 增量式旋转编码器 | 600 - 2000 PPR | 1 | 强烈建议必装 |
| 小车位置编码器 | 电机 encoder / 线性 encoder | 分辨率越高越好 | 1 | 可先用电机 encoder 换算 |
| 零点传感器 | 光电 / 霍尔开关 | 5V / 24V | 1 | homing 用 |
| 限位开关 | 微动 / 光电开关 | 两端限位 | 2-4 | 硬件保护 |
| 相机 | USB camera | 60 FPS 以上优先 | 1 | global shutter 更好 |
| 补光 | LED 灯条 / 面光源 | 固定亮度 | 1 | 降低视觉 domain gap |

---

### 8.4 控制与计算

| 模块 | 物料 | 建议规格 | 数量 | 备注 |
|---|---|---:|---:|---|
| MCU | STM32 / Teensy / ESP32 | 推荐 STM32 或 Teensy | 1 | 读 encoder、发 motor command |
| 上位机 | Laptop / Mini PC / Jetson | 有 GPU 更好 | 1 | 跑 LeWorldModel |
| 通信 | USB serial / CAN / UART | 先 USB serial | 1 | 简单可靠 |
| 数据盘 | SSD | 500GB+ | 1 | 图像数据占空间 |
| USB Hub | USB3.0 | 稳定供电 | 1 | 相机连接用 |

---

## 9. 数据格式建议

建议统一为 episode 格式：

```text
episode_000001/
├── rgb/
│   ├── 000000.png
│   ├── 000001.png
│   ├── 000002.png
│   └── ...
├── actions.npy
├── encoder_states.npy
├── timestamps.npy
├── dones.npy
├── metadata.json
└── calibration.json
```

---

## 9.1 单步数据字段

可以抽象成：

```json
{
  "t": 0.020,
  "image": "rgb/000001.png",
  "action": 0.42,
  "cart_pos": 0.123,
  "cart_vel": -0.018,
  "pole_angle": 0.31,
  "pole_ang_vel": 1.25,
  "done": false,
  "failure_reason": null
}
```

---

## 9.2 metadata 示例

```json
{
  "track_length_m": 1.0,
  "cart_mass_kg": 0.5,
  "pole_length_m": 0.4,
  "pole_mass_kg": 0.05,
  "camera_fps": 60,
  "control_dt": 0.02,
  "action_type": "normalized_pwm",
  "motor_driver": "closed_loop_stepper_driver",
  "failure_condition": "cart_limit_or_angle_limit"
}
```

---

## 10. Replay 工具很重要

采完数据后，先不要急着训练。

应该先写一个 replay 工具。

功能：

```text
读取 episode
播放图像序列
同步显示 action
同步显示 cart_pos 曲线
同步显示 pole_angle 曲线
标记 done / failure reason
```

这个工具后续会用于：

```text
检查数据质量
检查时间戳是否对齐
检查 action 是否正确记录
检查图像是否丢帧
检查 encoder 是否异常
可视化模型预测结果
对比真实 rollout 和模型 rollout
```

---

## 11. 关键 debug 指标

真实系统一定要记录延迟。

至少区分：

```text
camera exposure latency
camera transfer latency
image preprocessing latency
model inference latency
planner latency
USB / serial latency
MCU control loop latency
motor driver response latency
mechanical response latency
```

推荐记录：

```text
timestamp_image_captured
timestamp_image_received
timestamp_action_computed
timestamp_action_sent
timestamp_action_applied
timestamp_encoder_read
```

CartPole 是快系统，延迟会显著影响控制稳定性。

---

## 12. baseline controller

必须实现 classical baseline。

至少包括：

```text
PID
LQR
random policy
scripted sinusoidal policy
```

baseline 的作用不是替代 LeWorldModel，而是用于：

```text
验证硬件
安全保护
采集数据
提供性能下界
debug 模型控制结果
```

建议顺序：

```text
random / scripted action
    ↓
PID 控小车位置
    ↓
LQR 控倒立平衡
    ↓
world model planning
```

---

## 13. 第一阶段最小目标：M1

当前最建议的 milestone：

> M1：真实 CartPole 数据采集平台。

M1 不要求倒立成功。

M1 要求：

```text
1. 小车能左右移动
2. 摆杆能自由转动
3. cart position 可读
4. pole angle 可读
5. 相机画面稳定
6. action / image / encoder / timestamp 同步记录
7. 能保存 episode
8. 能 replay episode
9. 能用真实数据训练 LeWorldModel 做 one-step prediction
```

做到 M1 后，后面的任务会变得非常具体。

---

## 14. M1 之后的实验设计

### 14.1 数据采集实验

采集以下类型的数据：

```text
random action
small random action
sinusoidal action
step response
manual disturbance
near-failure cases
baseline controller data
```

不要只采平衡附近的数据。

world model 需要看到系统偏离稳定区、接近失败、摆杆大幅摆动、小车接近边界等情况。

---

### 14.2 模型对比实验

至少做三组：

```text
sim-only world model
real-only world model
sim pretrain + real finetune world model
```

比较：

```text
one-step prediction
multi-step rollout
真实控制中的表现
数据效率
泛化能力
```

---

### 14.3 sim-to-real 实验

需要记录仿真和真实的差异：

```text
轨道长度
小车质量
摆杆长度
摆杆质量
摩擦
电机响应
action scale
相机视角
光照
背景
帧率
控制延迟
```

可以逐步加入 domain randomization：

```text
mass randomization
length randomization
friction randomization
camera pose randomization
lighting randomization
background randomization
action noise
observation noise
latency randomization
```

---

## 15. 推荐 repo 结构

```text
cartpole-realworld/
├── docs/
│   ├── 00_project_goal.md
│   ├── 01_hardware_design.md
│   ├── 02_bom.md
│   ├── 03_isaaclab_env.md
│   ├── 04_data_format.md
│   ├── 05_world_model_training.md
│   ├── 06_real_deployment.md
│   ├── 07_experiments.md
│   └── 08_failure_cases.md
├── hardware/
│   ├── cad/
│   ├── stl/
│   ├── wiring/
│   └── calibration/
├── firmware/
│   ├── mcu_encoder_motor_control/
│   └── safety_supervisor/
├── isaaclab/
│   ├── cartpole_env/
│   └── domain_randomization/
├── data/
│   ├── README.md
│   └── dataset_schema.md
├── leworldmodel/
│   ├── train/
│   ├── eval/
│   └── deploy/
└── scripts/
    ├── collect_real_data.py
    ├── collect_sim_data.py
    ├── replay_dataset.py
    ├── evaluate_rollout.py
    └── deploy_policy.py
```

---

## 16. 当前最具体的下一步任务

接下来只做这几件事：

```text
任务 1：确定第一版硬件方案
任务 2：画出第一版机械结构图
任务 3：确定电机、导轨、编码器、相机
任务 4：写 dataset schema
任务 5：让 IsaacLab 导出的数据格式和真实数据格式一致
任务 6：写 replay 工具
任务 7：采集 10 个真实 random/scripted episode
任务 8：用真实数据训练 LeWorldModel 做下一帧预测
```

不要继续泛泛想“怎么部署世界模型”。

当前最小闭环是：

```text
真实 CartPole 随机动作采集 10 个 episode
    ↓
保存 image / action / state / timestamp
    ↓
用 replay 脚本播放
    ↓
用 LeWorldModel 预测下一帧
    ↓
评估预测误差
```

---

## 17. 总结

当前不是缺知识，而是需要一个收敛的工程目标。

当前阶段建议目标：

> 先做一个能稳定采集真实视觉、action、state 数据的 CartPole 实验平台。

项目主线应该是：

```text
真实数据平台
    ↓
真实世界模型
    ↓
sim-to-real 对比
    ↓
world model planning
    ↓
真实闭环部署
```

最重要的原则：

```text
最终可以 image-only，工程上必须有 encoder。
最终可以模型控制，第一版必须有 baseline。
最终可以强化学习，第一版先做数据采集。
最终可以追求平衡，当前先追求可观测、可复现、可安全失效。
```

一句话：

> 先把真实 CartPole 做成一个可靠的数据采集和实验平台，而不是一开始就做成一个复杂的智能控制系统。
