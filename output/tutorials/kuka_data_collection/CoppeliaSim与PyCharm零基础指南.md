# CoppeliaSim 与 PyCharm 零基础指南

从打开 KUKA 模型，到亲手采集第一份数据

**为你当前的环境编写：Windows · Python 3.12.1 · PyCharm**

修订日期：2026-10-16。练习场景来自本机 CoppeliaSim Edu 4.10.0 的内置 KUKA LBR iiwa 14 R820 模型，是七轴机械臂。以后确定论文或实机的具体型号后，再更换相应模型。

### 这次先完成什么

| 阶段 | 你要完成的事 | 到这里就算成功 |
| --- | --- | --- |
| 第一课：软件与连接 | 打开场景、打开 PyCharm 项目、选择解释器、运行连接检查 | 终端出现 `[OK] Connected to CoppeliaSim.` |
| 第二课：第一份数据 | 选择基座与末端、采 100 条数据、抽查回放 | 出现 `[OK] CSV validation passed.` |
| 后续：训练准备 | 扩大到 10,000 条，再接已有 CSV 训练器 | 有独立保存、校验通过的 KUKA 数据 |

**第一次只需按顺序完成第一课。** 不需要先理解全部 API，也不用先跑一百万条数据。

配套文件夹已经准备好：

```text
D:\Swansea-杜亦冯\biyeshj\output\tutorials\kuka_data_collection
```

本机已建立独立 `.venv`，不会改动你正在使用的 TIAGo 训练环境。本指南末尾有重建环境的方法，只有换电脑、移动项目或环境缺失时才需要用。

### 已经替你验证的部分

已在独立的无窗口 CoppeliaSim 实例中，用这套采集函数生成 100 条七轴数据，并回放其中 10 条：位置和朝向残差均为 0，原始关节角已恢复。记录见 `验证记录.json`。这验证的是采集与回放的一致性；PyCharm 的界面点击仍需你按本指南完成。

<!-- pagebreak -->

## 1. 先分清：哪个软件负责什么

| 名称 | 你可以怎样理解 | 本次作用 |
| --- | --- | --- |
| CoppeliaSim | 电脑里的虚拟机械臂 | 改关节角，读取末端位置与朝向 |
| PyCharm | 写代码、选 Python、运行代码的工作台 | 管理这个采集项目 |
| Python 解释器 | 真正执行 `.py` 文件的程序 | 本项目用 Python 3.12.1 |
| PowerShell Terminal | 输入系统命令的终端 | 输入 `python.exe 某脚本.py` |
| `.venv` | 这个项目自己的 Python 环境 | 保存连接 CoppeliaSim 所需的库 |

### 一条数据是怎样来的

```text
Python 随机产生 7 个关节角
           ↓
CoppeliaSim 按这组角度摆好 KUKA
           ↓
读取实际关节角 + 末端相对基座的位姿
           ↓
写入 CSV，成为一条训练样本
```

训练时把方向反过来：给神经网络末端位姿，让它预测关节角。这就是你现在研究的逆运动学任务。

### 文件后缀速查

| 后缀 | 内容 | 怎么打开 |
| --- | --- | --- |
| `.ttt` | 一个完整 CoppeliaSim 场景 | CoppeliaSim 的 File > Open Scene |
| `.ttm` | 一个可放进场景的模型 | CoppeliaSim 的 File > Load model |
| `.py` | Python 代码 | PyCharm 编辑；由 Python 执行 |
| `.csv` | 一行一条样本的数据表 | 可用表格软件查看；训练器读取 |
| `.json` | 配置或检查结果 | PyCharm 中打开阅读 |
| `.cmd` | Windows 启动脚本 | 资源管理器双击 |

<!-- pagebreak -->

## 2. 第一课 A：打开准备好的场景

### 情况一：CoppeliaSim 还没有打开

**在哪里：Windows 文件资源管理器。**

1. 打开本指南第 1 页给出的 `kuka_data_collection` 文件夹。
2. 找到 `start_coppelia.cmd`，双击一次。
3. 等待 CoppeliaSim 打开。启动器会指定本项目的 Python，并加载练习场景。
4. 在三维视图中应看到一台 KUKA 机械臂；左侧场景树应有 `LBRiiwa14R820`。

启动器按本机已安装路径编写。如果提示找不到 CoppeliaSim，按下面的手动打开方法操作，并查看第 14 节。

### 情况二：CoppeliaSim 已经打开

**在哪里：CoppeliaSim 顶部菜单。** 不必再双击启动器。

1. 如果当前场景需要保留，先保存。
2. 点击 **File > Open Scene…**。
3. 进入本项目的 `scenes` 文件夹，选中 `kuka_beginner_verified.ttt`，点击打开。
4. 确认出现 `LBRiiwa14R820`。第一次练习只开一个 CoppeliaSim 窗口，连接端口更容易判断。

场景完整路径：

```text
D:\Swansea-杜亦冯\biyeshj\output\tutorials\kuka_data_collection\scenes\kuka_beginner_verified.ttt
```

### 暂时不要点击 Play

本教程采集的是**仿真停止时的静态运动学数据**。保持停止状态即可。若已经点了 Play，点击方形的 **Stop** 按钮并等待结束；**Pause 暂停不等于 Stop 停止**。

**本步成功标志：**场景里有机械臂，左侧有对应模型，仿真没有运行。

<!-- pagebreak -->

## 3. 认识场景树，避免选错对象

**在哪里：CoppeliaSim 左侧的场景层级树。** 这里列出当前场景中的对象，不是旁边用于浏览模型缩略图的模型库。

### 现在只学三个动作

| 动作 | 操作方法 | 用途 |
| --- | --- | --- |
| 展开对象 | 点击名称左侧的小三角 | 查看下面的关节和连杆 |
| 选择对象 | 单击名称一次 | 告诉脚本你选的是哪个对象 |
| 移动视角 | 按工具栏的平移、旋转或缩放工具操作 | 看清机械臂；不影响采集定义 |

选择时不要按 Ctrl 或 Shift，不要拖动对象名称。双击名称可能进入改名；这次用单击即可。

### 本练习要找到的两个对象

**基座：`LBRiiwa14R820`。末端：最后一个关节下面的 `link8_resp`。**

下面只显示主链，实际场景还有其他分支。逐层展开，不要把所有叫 `joint` 的对象当成同一个关节。

```text
LBRiiwa14R820             ← 基座：选这一行
  joint
    link2_resp → joint → link3_resp → joint
      → link4_resp → joint → link5_resp → joint
        → link6_resp → joint → link7_resp → joint
          → link8_resp    ← 末端：选这一行
```

这条链有 **7 个关节**。`14 R820` 中的“14”不是关节数。

`link8_resp` 的对象坐标系是本次练习使用的末端参考系，并不自动等于真实机器人的法兰中心或夹爪工具点 TCP。以后加夹爪时，需要重新定义末端并重新采集数据。

**本步成功标志：**能在左侧树中找到基座和 `link8_resp`。现在不用保存任何配置。

<!-- pagebreak -->

## 4. 第一课 B：用 PyCharm 打开项目

**在哪里：PyCharm。** 你原来的 TIAGo 训练可以继续保留。

1. 点击左上角主菜单，找到 **File > Open…**。欢迎页也有 Open。
2. 选择整个 `kuka_data_collection` **文件夹**，不要只选某一个 `.py` 文件。
3. 如果询问在哪里打开，选择 **New Window**，让 KUKA 与 TIAGo 各用一个窗口。
4. 等待左侧 Project 文件列表显示出来。

### 应该看到的文件

```text
kuka_data_collection
  .venv
  scenes
    kuka_beginner_verified.ttt
  00_check_connection.py
  01_select_frames.py
  02_collect_static.py
  03_validate_csv.py
  coppelia_helpers.py
  requirements.txt
  start_coppelia.cmd
```

`.venv` 有时显示为 library root 或被折叠，这不影响使用。之后还会生成 `data` 文件夹、连接报告和配置文件。

### 不要把文件夹作为 Python 包安装

这是可直接运行的脚本项目。你之前出现的 `pip install -e …` 和 “neither setup.py nor pyproject.toml found” 是把普通文件夹当成了可安装包。

本项目只需选择正确解释器。依赖安装使用 `-m pip install -r requirements.txt`，不用 `pip install -e`。

**本步成功标志：**PyCharm 左侧能看到 `00_check_connection.py` 等文件，窗口对应的是 KUKA 项目。

<!-- pagebreak -->

## 5. 选对 Python 解释器

**在哪里：PyCharm 设置。** 本项目虚拟环境已经在你的电脑上准备好。

1. 按 **Ctrl + Alt + S** 打开 Settings。
2. 在设置搜索框输入 **Python Interpreter**，打开项目解释器页面。
3. 找到 **Add Interpreter > Add Local Interpreter**。不同版本的按钮名称可能稍有区别。
4. 选择已有环境 / Existing environment，或选择已有 Python 可执行文件；不要再新建第二个 `.venv`。
5. 浏览并选中下面这个 `python.exe`，然后 Apply / OK。

```text
D:\Swansea-杜亦冯\biyeshj\output\tutorials\kuka_data_collection\.venv\Scripts\python.exe
```

### 如何确认自己选对了

| 检查位置 | 应该是什么 |
| --- | --- |
| Interpreter path | 路径里有 `kuka_data_collection\.venv\Scripts\python.exe` |
| Python 版本 | Python 3.12.1；显示为 3.12 也正常 |
| 包列表 | 有 `coppeliasim-zmqremoteapi-client`、`pyzmq`、`cbor` |

不要只看“Python 3.12”几个字，因为 TIAGo 项目的解释器也是 3.12。**文件路径才决定你使用的是哪个环境。**

### 如果暂时找不到设置入口

下一节的终端命令直接指定 `.venv\Scripts\python.exe`，仍然可以运行。之后再设置解释器，以便使用 PyCharm 的绿色运行按钮。

**本步成功标志：**解释器路径指向 KUKA 项目的 `.venv`。如果这个文件不存在，转到第 14 节重建环境。

<!-- pagebreak -->

## 6. 打开 Terminal，先检查环境

**在哪里：PyCharm 下方的 Terminal 终端。** 如果找不到，使用 **View > Tool Windows > Terminal**；常用快捷键是 Alt + F12。

这里应该有包含 **`PS` 和 `>`** 的 PowerShell 提示符。前面的 `(.venv)` 可能有，也可能没有；本指南的命令指定了完整的项目 Python，因此都能按同一种方式运行。

### 先切换到项目文件夹

下面两行在 **PowerShell Terminal** 中逐行粘贴，每行按一次 Enter：

```powershell
Set-Location -LiteralPath 'D:\Swansea-杜亦冯\biyeshj'
Set-Location -LiteralPath 'output\tutorials\kuka_data_collection'
```

只复制代码框中的命令，不复制 `PS D:\...>` 提示符。

### 再执行两个检查

```powershell
.\.venv\Scripts\python.exe --version
.\.venv\Scripts\python.exe -m pip check
```

应分别看到 `Python 3.12.1` 和 `No broken requirements found.`。

### 你之前遇到的 import 报错，怎么避免

| 看到的界面 | 可以输入什么 |
| --- | --- |
| `PS … >` | 系统命令，例如 `python.exe 00_check_connection.py` |
| Python 文件编辑区 | `import numpy as np` 等 Python 代码 |
| Python Console 的 `>>>` | Python 代码；本教程暂时不用这个窗口 |

**不要把 `import ...` 或 `from ... import ...` 直接粘贴到 PowerShell。** 本教程已经把 Python 代码写进 `.py` 文件，你只需执行运行文件的命令。

<!-- pagebreak -->

## 7. 第一课 C：运行连接检查

### 推荐方法：在 Terminal 运行

确认 CoppeliaSim 已打开练习场景，再在 **KUKA 项目的 PowerShell Terminal** 输入：

```powershell
.\.venv\Scripts\python.exe 00_check_connection.py
```

正常输出包含：

```text
[OK] Connected to CoppeliaSim.
```

还会列出场景里的关节信息，并在项目文件夹生成 `connection_report.json`。练习场景应有 7 个机械臂关节。第一次先核对这个数量，不用理解每一项 JSON。

这一步只读取场景信息，不会让仿真开始运行。默认连接本机 `localhost:23000`。

### 另一种方法：右键运行当前文件

1. 在 PyCharm 左侧 **右键 `00_check_connection.py`**。
2. 选择 **Run '00_check_connection'**。
3. 在下方 Run 窗口查看输出。
4. 看窗口第一行：它应使用 KUKA 的 `.venv\Scripts\python.exe`，并执行 `00_check_connection.py`。

**打开某个代码标签页，不一定会改变顶部绿色三角的运行目标。** 你之前看 `.npz` 时仍运行了 `01_warmup_numpy.py`，就是这个原因。初学时右键具体文件，或直接用本指南的终端命令。

### 今天可以先停在这里

**第一课完成标志：**连接成功、关节数量正确、没有红色错误。下一课再开始选择坐标系和采集数据。

如果连接失败，查看第 13 节，先排查软件是否打开、是否多开和端口。

<!-- pagebreak -->

## 8. 第二课 A：保存基座与末端配置

**准备：CoppeliaSim 保持 Stop；PyCharm Terminal 位于 KUKA 项目。**

在终端输入：

```powershell
.\.venv\Scripts\python.exe 01_select_frames.py
```

### 第一次选择：基座

1. 终端出现 **`STEP 1`** 后，切换到 CoppeliaSim。
2. 在左侧场景树中单击 **`LBRiiwa14R820`** 这一行。只选一个对象。
3. 切回 **PyCharm 的 Terminal**，点击正在等待的输入行，按 **Enter**。
4. 终端应显示 `Selected: /LBRiiwa14R820`，然后出现 `STEP 2`。

### 第二次选择：末端

1. 切换回 CoppeliaSim，逐层展开主链。
2. 找到并单击最后一个关节下面的 **`link8_resp`**。
3. 切回 Terminal，再按一次 **Enter**。

成功时会看到：

```text
[OK] Found the ordered arm chain: 7 joints.
Saved: model_config.json
```

脚本会沿真实父子层级找到 7 个关节并排序。你不用手工给关节改名，也不用把很长的对象路径抄进代码。

### 下次是否还要运行 01

同一场景、基座、末端、关节层级和限位没变，就可直接复用 `model_config.json`。重新运行时提示配置已存在，是保护已有配置；跳到下一节即可。

换模型或改末端时，另存新配置，例如加 `--out model_config_v2.json`；之后采集要带上 `--config model_config_v2.json`。第一次练习先保持默认名称。

**本步成功标志：**找到 7 个关节，并生成 `model_config.json`。

<!-- pagebreak -->

## 9. 第二课 B：只采 100 条

**在哪里：PyCharm 的 PowerShell Terminal。** CoppeliaSim 保持打开、停止；采集期间不要切换场景或点击 Play。

```powershell
.\.venv\Scripts\python.exe 02_collect_static.py --samples 100
```

采集期间机械臂姿态可能快速变化。这是脚本在设置不同关节角，不需要你手动拖动机械臂，也不需要打开仿真。

### 成功时会看到什么

```text
Collected 100/100
[OK] CSV rows: 100
Saved: kuka_static_100_01.csv
Original joint angles restored: True
```

初次运行会在本项目的 `data` 文件夹生成：

| 文件 | 用途 |
| --- | --- |
| `kuka_static_100_01.csv` | 100 条关节角与末端位姿 |
| `kuka_static_100_01.meta.json` | 采集状态、模型配置、限位、随机种子、文件校验值 |

结束后，脚本会尝试恢复采集前的关节角。只有输出 `True` 才表示恢复检查通过。若中途关闭软件或断开连接，可能无法恢复；重新打开练习场景再继续。

### 一行 CSV 包含什么

| 列 | 意义 | 单位或顺序 |
| --- | --- | --- |
| `arm_1` 到 `arm_7` | 按基座到末端顺序排列的实际关节角 | 弧度 rad |
| `ee_x, ee_y, ee_z` | 末端相对基座的位置 | 米 m |
| `ee_quat_x` 到 `ee_quat_w` | 末端相对基座的朝向四元数 | x、y、z、w |

这里的四元数顺序是 **XYZW**。不要套用之前 TIAGo 原始文件的列名调整方式。

**本步成功标志：**写入 100 行、保存 CSV 和元数据、角度恢复为 True。100 条用于检查流程，尚不足以做完整训练结论。

<!-- pagebreak -->

## 10. 第二课 C：检查并回放数据

**先确认上一节真的生成了 CSV，再执行这一节。** 首次采集使用默认文件名时，在 Terminal 输入：

```powershell
.\.venv\Scripts\python.exe 03_validate_csv.py --replay
```

`--replay` 会把随机抽取的 10 组 CSV 关节角放回同一个场景，重新读取末端位姿。CoppeliaSim 必须仍然打开且处于停止状态。

### 成功输出怎样理解

```text
[OK] CSV validation passed.
```

| 输出字段 | 你应怎样理解 |
| --- | --- |
| `rows: 100` | 读取了 100 条数据 |
| `joint_count: 7` | 七轴关节标签 |
| `input_size: 7` | 输入为位置 3 维 + 四元数 4 维 |
| `poses_replayed: 10` | 抽查了 10 条 |
| `max_position_residual_m` | 原记录与回放位置的最大差值 |
| `max_orientation_residual_deg` | 原记录与回放朝向的最大差值 |

验证器要求位置残差不大于 0.00001 m，朝向残差不大于 0.001°。本机实测为 0；这只说明同一模型下采集和回放一致。

它也会检查列名、有限数值、关节限位、单位四元数，以及 CSV 是否匹配采集元数据。检查结果保存在同名 `.validation.json`。

### 如果你已经采了第二次

采集器会自动使用 `_02`、`_03` 等新名称，保护旧数据。检查第二次的文件时明确指定：

```powershell
.\.venv\Scripts\python.exe 03_validate_csv.py --csv data/kuka_static_100_02.csv --replay
```

**第二课完成标志：**CSV 检查与回放通过。这里还没有训练神经网络，也没有做碰撞筛选。

<!-- pagebreak -->

## 11. 流程通了，再扩大到 10,000 条

### 第一步：采集和检查

**在 KUKA 项目的 PowerShell Terminal：**

```powershell
.\.venv\Scripts\python.exe 02_collect_static.py --samples 10000
.\.venv\Scripts\python.exe 03_validate_csv.py --csv data/kuka_static_10000_01.csv --replay
```

第二行的文件名要与第一行实际生成的文件一致。数据量增加后采集会更久，以终端进度为准；采集期间保持场景不变。

### 第二步：用已有 CSV 训练器检查列

你原来的 `02_train_pose_csv.py` 支持这套 CSV 列。**这一段回到 TIAGo 项目的 PowerShell Terminal**，明确使用旧训练环境：

```powershell
Set-Location -LiteralPath 'D:\Swansea-杜亦冯\biyeshj'
Set-Location -LiteralPath 'output\tutorials\tiago_ik_beginner'
$kukaCsv = '..\kuka_data_collection\data\kuka_static_10000_01.csv'
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv $kukaCsv --check-only
```

应看到 `x_shape` 为 `[10000, 7]`、`y_shape` 为 `[10000, 7]`，前提是你的 CSV 确有 10,000 条且无重复行被移除。

检查通过后，可接着运行下面一行。输出文件夹必须是新名称：

```powershell
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv $kukaCsv --out runs/kuka_01
```

### 这一阶段先比较什么

先把数据来源、模型、基座、末端、关节顺序固定，再比较网络层数。七轴逆运动学可能一对多；标签角度误差高，不一定只是层数不足。

训练输出中的关节 MAE 不等于末端任务成功率。你之前的 `04`、`06`、`07` 脚本含有 TIAGo 专用正运动学，**不能直接用于 KUKA 位姿评价**。下一阶段需接入同一 KUKA 模型，评估预测角度对应的位置、朝向、限位和碰撞。

<!-- pagebreak -->

## 12. 换成自己下载的 KUKA 模型

前面的练习通过后，再用 GitHub 上的场景或其他现成模型。开始时记录准确型号、来源和版本。

| 下载到的内容 | 在 CoppeliaSim 里的操作 | 注意什么 |
| --- | --- | --- |
| `.ttt` 场景 | File > Open Scene… | 会打开完整场景，可能有输送带、夹爪和控制脚本 |
| `.ttm` 模型 | File > New Scene，再 File > Load model… | 加载到新场景后，另存为自己的 `.ttt` |
| URDF 等描述文件 | 使用对应导入功能 | 这不属于本次最短入门路线；先用已验证场景 |

### 如果想自己加载本机内置模型

通过 **File > Load model…** 选择：

```text
C:\Program Files\CoppeliaRobotics\CoppeliaSimEdu\models\robots\non-mobile\KUKA LBR iiwa 14 R820.ttm
```

加载前新建场景，避免重复放入两台同名机器人。加载后用 **File > Save Scene as…** 保存练习场景。

### 对新模型重新做哪些检查

1. 保持停止状态，重新运行 `00_check_connection.py`，查看关节数量。
2. 找到固定基座和最后一个机械臂关节之后的末端对象，排除夹爪手指关节。
3. 重新运行 `01_select_frames.py`，另存配置；六轴模型还需加 `--dof 6`。
4. 先采 100 条、检查回放，再扩大数据量。不要与旧模型的 CSV 混用。

你截图里的仓库使用较老的 `sim.py / remoteApi.dll` 接口。这个指南采用当前的 ZeroMQ 接口；可以研究其模型和场景，但不要把旧 DLL 直接复制到本项目当作连接方法。

**迁移前必须确定：**具体 KUKA 型号、关节顺序、角度限位、基座坐标系、末端工具坐标系。静态采集没有模拟真实控制延迟，也没有自动排除自碰撞。

<!-- pagebreak -->

## 13. 报错时先看这张表

| 现象或提示 | 常见原因 | 具体操作 |
| --- | --- | --- |
| PowerShell 不认识 `import` / `from` | 把 Python 代码输入了系统终端 | 不粘贴 Python 语句；运行本指南的 `python.exe 文件.py` 命令 |
| 仍在运行旧 warmup 脚本 | 顶部运行配置仍指向旧脚本 | 右键本项目的目标文件运行，或复制本指南终端命令 |
| `No interpreter` | PyCharm 没选择 Python | 按第 5 节选择 KUKA 的 `.venv\Scripts\python.exe` |
| 找不到项目 `python.exe` | 文件夹不对或环境不存在 | 先按第 6 节切换目录；环境确实缺失时按第 14 节重建 |
| `ModuleNotFoundError` | 用了另一个 Python，或依赖没装 | 核对运行第一行的解释器路径，再执行本项目依赖安装命令 |
| `neither setup.py nor pyproject.toml` | 执行了 `pip install -e` | 改用 `-m pip install -r requirements.txt` |
| `CoppeliaSim is not reachable` | 软件没开，或连接端口不对 | 打开软件和练习场景；默认使用 localhost:23000，先只保留一个实例 |
| `No valid ZeroMQ API response` | 端口对应的服务无响应 | 查看 CoppeliaSim 日志及 ZeroMQ 服务；确认不是旧 Legacy API 接口 |
| `Click STOP… PAUSE is not STOP` | 仿真运行或只是暂停 | 点击 Stop，等待停止后重试 |
| `Select exactly one object` | 没选对象或多选了 | 在场景树单击一行，不按 Ctrl / Shift，再重新运行配置脚本 |
| `Found 6 arm joints, expected 7` | 选中了最后的关节本身 | 展开最后关节，选它下面的 `link8_resp`，再重试 |

排查时先看终端最后几行，再向上看运行命令。不要只根据红色文字的颜色判断整个环境坏了。

<!-- pagebreak -->

## 14. 文件、环境与恢复方法

| 提示或情况 | 你应怎样处理 |
| --- | --- |
| 找不到 `model_config.json` | 先完成第 8 节的两次对象选择 |
| 配置文件已经存在 | 同一场景复用它；换场景用新的 `--out` 配置名 |
| 找不到 `data/...csv` | 先完成采集，再按实际生成的文件名运行验证；示例路径不会自动产生文件 |
| CSV 或元数据已存在 | 不指定 `--out`，让采集器自动编号；无需删除旧数据 |
| CSV 不匹配元数据 | 不要用表格软件改写训练原件；从未修改的备份验证，或重新采集 |
| 回放残差超限 | 确认模型、基座、末端和场景没变；用同一配置重新采集 |
| CoppeliaSim 启动报 Python 缺 `zmq` | 保存需要的场景，关闭闲置实例；改用配套 `start_coppelia.cmd` 指向本项目环境 |

### 只有环境缺失、移动项目或换电脑时：重建环境

**在哪里：KUKA 项目文件夹的 PowerShell Terminal。** 本机已有环境时不用再执行。复制压缩包到别处时，虚拟环境应在新位置重新建立。

下面第一行使用你本机现有的 Python 3.12.1 安装路径：

```powershell
& 'C:\Program Files\Python312\python.exe' -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
```

不需要执行激活脚本。命令中的 `&` 用于调用带空格路径的程序，请保留。换电脑时，如果 Python 安装在别处，第一行需使用那台电脑的真实路径。

### 中途想停止采集

在运行采集的 Terminal 按 **Ctrl + C**，等待脚本处理退出。脚本会尽力恢复角度，并保留已写入文件，元数据标为未完成；这份文件不会通过完整性检查。下次重跑会使用新编号。

如果连接已经断开，直接重新打开准备好的练习场景，再按流程继续。

<!-- pagebreak -->

## 15. 完成清单与查阅入口

### 第一课完成清单

- CoppeliaSim 中打开的是练习 KUKA 场景，仿真保持停止。
- PyCharm 打开了 `kuka_data_collection` 文件夹。
- 解释器使用本项目 `.venv\Scripts\python.exe`。
- 运行 00 后出现连接成功，并看到 7 个机械臂关节。

### 第二课完成清单

- 配置保存的基座为 `LBRiiwa14R820`，末端为 `link8_resp`。
- 采集得到 100 行 CSV 和匹配的 `.meta.json`，原始角度恢复为 True。
- 运行 03 后检查通过，10 条回放残差满足阈值。
- 知道这份数据没有碰撞筛选，也不是神经网络预测结果。

### 常用文件位置

| 要找什么 | 在哪里 |
| --- | --- |
| 所有配套脚本、可复制命令 | `output/tutorials/kuka_data_collection` |
| 准备好的练习场景 | 项目中的 `scenes` 文件夹 |
| 你亲自采集的 CSV | 项目中的 `data` 文件夹，首次采集后生成 |
| 对象路径与限位配置 | 项目中的 `model_config.json`，运行 01 后生成 |
| 本指南 PDF | `output/pdf/CoppeliaSim与PyCharm零基础指南.pdf` |

### 官方资料

- [CoppeliaSim：场景的打开与保存](https://manual.coppeliarobotics.com/en/scenes.htm)
- [CoppeliaSim：模型的加载](https://manual.coppeliarobotics.com/en/models.htm)
- [CoppeliaSim：界面与场景层级树](https://manual.coppeliarobotics.com/en/userInterface.htm)
- [ZeroMQ Python 接口](https://manual.coppeliarobotics.com/en/zmqRemoteApiOverview.htm)
- [末端位姿与四元数顺序](https://manual.coppeliarobotics.com/en/sim/simGetObjectPose.htm)
- [关节限位返回值](https://manual.coppeliarobotics.com/en/sim/simGetJointInterval.htm)
- [PyCharm：打开项目](https://www.jetbrains.com/help/pycharm/open-projects.html)
- [PyCharm：设置 Python 解释器](https://www.jetbrains.com/help/pycharm/configuring-python-interpreter.html)
- [PyCharm：运行 Python 文件](https://www.jetbrains.com/help/pycharm/code-running-assistance-tutorial.html)

命令优先从这份 Markdown 复制，避免 PDF 复制时出现换行或符号变化。界面菜单以你安装的版本为准；找不到解释器设置时，用 Ctrl + Alt + S 后搜索 Python Interpreter。
