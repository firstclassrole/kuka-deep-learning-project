# 用现成 KUKA 模型采集逆运动学训练数据

本指南先做与目前 TIAGo 练习相同的静态任务：末端位置和朝向作为输入，关节角作为输出。采集时从关节角出发，用仿真模型的正运动学生成配对位姿。

## 1. 选定具体型号

已在你电脑中确认 CoppeliaSim Edu 及这些现成模型：

- `C:\Program Files\CoppeliaRobotics\CoppeliaSimEdu\models\robots\non-mobile\KUKA LBR iiwa 7 R800.ttm`
- `C:\Program Files\CoppeliaRobotics\CoppeliaSimEdu\models\robots\non-mobile\KUKA LBR iiwa 14 R820.ttm`
- 同目录还提供 `KUKA LBR4+.ttm`。

如果毕业设计尚未固定 KUKA 型号，可先用内置 iiwa 14 R820 练习。若最后要对接学校的实体机器人，应优先采用实体机器人对应的型号、关节限位和工具配置。iiwa 7 R800 与 iiwa 14 R820 均为七轴，型号中的 7 和 14 对应额定负载，而不是关节数量。依据：[KUKA 官方产品资料](https://www.kuka.com/en-in/products/robotics-systems/industrial-robots/lbr-iiwa)。

截图中的 [CoppeliaSim-Kuka-Arm-Simulation](https://github.com/Ashwin-Vinoo/CoppeliaSim-Kuka-Arm-Simulation) 包含完整分拣场景与旧 Python 接口。它可以作为场景参考；第一轮静态数据采集只需要机械臂及工具，不需要传送带、相机与分拣流程。若选用第三方模型，需要记录来源、版本，并核对尺寸和关节限制。

## 2. 建立最小采集场景

打开 CoppeliaSim，新建空场景，加载选定的 `.ttm` 模型，另存一个专门的数据采集场景，例如 `kuka_dataset_scene.ttt`。先不要启动旧场景的自动抓取脚本。

确认三个对象定义：

| 对象 | 应如何确定 |
|---|---|
| 基坐标 base | 固定在机械臂底座的坐标系，可使用明确的底座对象或固定 dummy；不要选会随关节转动的对象 |
| 手臂关节 joints | 从底座到末端依次确认 A1…A7；夹爪手指关节不算机械臂的七个关节 |
| 末端坐标 tip/TCP | 明确使用末端法兰还是夹爪抓取点；必要时建立一个固定在法兰下的 dummy，并记录工具偏移 |

保存模型文件、场景文件及上述对象的实际路径。模型看起来相似，不代表尺寸、轴方向、零位和 TCP 一致。

## 3. 建一个独立的采集 Python 环境

目前的 TIAGo 网络可以继续运行。采集工作放在这个新目录，使用单独虚拟环境。在新的 PowerShell 终端执行：

```powershell
Set-Location -LiteralPath 'D:\Swansea-杜亦冯\biyeshj\output\tutorials\kuka_data_collection'
& 'C:\Program Files\Python312\python.exe' -m venv .venv
.\.venv\Scripts\python.exe -m pip install coppeliasim-zmqremoteapi-client
```

官方当前推荐的 ZeroMQ Remote API 支持 Python 从外部控制仿真与读取场景对象；旧 legacy remote API 已弃用。依据：[官方 ZeroMQ 接口说明](https://manual.coppeliarobotics.com/en/zmqRemoteApiOverview.htm)、[接口版本说明](https://manual.coppeliarobotics.com/en/remoteApiOverview.htm)。本指南未替你执行安装或更改当前训练环境。

## 4. 先做只读连接检查

保持 CoppeliaSim 程序打开，加载刚才的机械臂场景，然后在新目录的 PowerShell 中运行：

```powershell
.\.venv\Scripts\python.exe 00_check_connection.py
```

脚本只读取仿真状态、场景中的关节路径、关节限位和当前数值，不启动仿真、不改关节位置。场景中如果有其他设备或夹爪，也会列出它们的关节。输出顺序不能直接当作 A1…A7 顺序，需要沿机械链核对。

脚本已经过语法检查，尚未在你的实际仿真场景连接运行。连接不成功时，先确认程序已打开、ZeroMQ add-on 可用、客户端与仿真版本匹配。默认端口为 23000；如果本机改过端口，可通过 `--port` 指定。

## 5. 选择静态 FK 采集方式

第一轮推荐纯运动学采集，避免把动态控制误差混入训练标签。可以在停止仿真的静态场景中设置关节配置，再读取末端位姿；若在运行的仿真中采集，需确认关节处于合适的运动学模式且自动控制脚本不会覆盖配置。

官方说明运动学模式允许通过 `sim.setJointPosition` 直接改变位置，动态模式由物理引擎处理。依据：[关节模式](https://manual.coppeliarobotics.com/en/jointModes.htm)、[设置关节位置](https://manual.coppeliarobotics.com/en/sim/simSetJointPosition.htm)。

每条数据执行：

1. 读取每个手臂关节的限位，统一使用弧度。
2. 在限位内随机采样关节角，适当留出边界余量，固定随机种子。
3. 设置整条机械臂的配置。
4. 读回实际关节角，确认与设置值一致。
5. 读取末端相对 base 的位置和四元数。
6. 检查数值有限、四元数单位长度、关节范围和位姿合理性；按实验需要进行碰撞筛选。
7. 将实际角度与同一配置下的位姿写成 CSV 一行。

`sim.getJointInterval` 返回的是“下限、跨度”，上限要计算为 `下限 + 跨度`；循环关节需单独约定角度范围，不能把它的区间参数当作有限关节限位。依据：[关节限位接口](https://manual.coppeliarobotics.com/en/sim/simGetJointInterval.htm)。

若采用动态采集，应使用目标角驱动，按同步步进等待关节到位，再保存读回的实际角度。不能把尚未到达的目标角与当前末端位姿配成标签。

## 6. CSV 应保存什么

七轴模型的格式可与通用训练器保持一致：

```text
arm_1,arm_2,arm_3,arm_4,arm_5,arm_6,arm_7,ee_x,ee_y,ee_z,ee_quat_x,ee_quat_y,ee_quat_z,ee_quat_w
```

六轴模型则只有 `arm_1` 到 `arm_6`。位置采用米，角度采用弧度，所有位姿必须采用同一 base 和 TCP。

`sim.getObjectPose(tip, base)` 默认返回 `[x,y,z,qx,qy,qz,qw]`。这个四元数顺序与列名一致，不要套用之前 TIAGo Pykin 数据的特殊分量重排。依据：[读取位姿接口](https://manual.coppeliarobotics.com/en/sim/simGetObjectPose.htm)。

配套元数据至少记录：机器人型号、模型与场景文件及其校验值、来源 URL、CoppeliaSim/客户端版本、关节顺序与限位、base/TCP 对象路径和工具偏移、单位、四元数顺序、采样分布、随机种子、拒绝样本数量以及碰撞检查状态。

如果没有完成碰撞筛选，应明确写“未检查碰撞”。若筛选自碰撞，要合理设置需要检查的连杆对，避免相邻连杆的正常连接被错误计为碰撞。不要将未检查的静态构型称作可执行轨迹。碰撞接口参考：[sim.checkCollision](https://manual.coppeliarobotics.com/en/sim/simCheckCollision.htm)。

## 7. 先验证 100 条，再扩大数据量

先采 100 条并人工检查若干配置：角度与末端显示是否一致，零位/轴方向是否正确，重新设置 CSV 中的关节角能否重现对应位姿。

通过后再采 10,000 条作为基线，随后按需要扩大到 100,000 条。均匀采关节角不等于在笛卡尔工作空间均匀采样；需要查看工作空间覆盖。七轴冗余还会带来多解，数据量增加不保证直接回归关节标签就达到高末端精度。

训练前去重并固定训练/验证/测试划分，缩放只在训练集拟合。若后来采的是连续轨迹，应按轨迹或场景划分，而不是随机拆散相邻帧。

## 8. 接入训练与 KUKA 评估

现有 `tiago_ik_beginner/02_train_pose_csv.py` 是通用 CSV 训练器，可读取合规的 KUKA 六轴或七轴 CSV。训练器能复用，但训练数据和模型输出必须对应同一个机器人。

`04_evaluate_tiago.py`、`06_refine_tiago.py`、`07_run_depth.py` 的 FK 或入口检查当前绑定 TIAGo 模型，不能用于 KUKA 的末端评估。KUKA 后续应在同一 CoppeliaSim 模型中设置网络预测角，读出末端位姿，与目标比较，或使用经过该模型逐点校验的 KUKA URDF/FK。

最终至少报告末端位置平均误差与 P95、朝向角误差、关节限位，以及明确的碰撞/阈值检查范围。单独的关节标签 MAE 仍不能代表机械臂到达目标的成功率。

本流程生成静态逆运动学数据。如果毕业设计进一步要预测未来角度或研究动态控制，则还需要时间戳、连续轨迹、速度/加速度、控制输入和采样周期，不能直接用这份静态表代替时序数据。
