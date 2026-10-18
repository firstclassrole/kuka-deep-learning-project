# 神经网络逆运动学新手教程

当前版本适配你已跑通的 **Windows、PyCharm、Python 3.12.1、NumPy 2.5.4**。先阅读 `第一篇论文新手教程_数据准备修订版.pdf`；复制命令时使用 `第一篇论文新手教程.md`。已有的 `.venv` 可以继续使用。修订版补充了真实数据准备的前提，并将可直接运行的 CSV 示例改为配套的二维数据。

**新增：两层、十层和作者 80 层隐藏结构的运行入口见 `明天运行说明.md`。** 直接运行 `07_run_depth.py --preset two`、`--preset ten` 或 `--preset author80`，自动保存到新目录并评估验证集。作者 80 层入口是 NumPy 结构适配版；完整原源码另存于 `references/author_v1/source`，尚未配置原版 TensorFlow 环境。

## 选择现有解释器

在 PyCharm 的 Python Interpreter 设置中选择项目目录下的 `.venv\Scripts\python.exe`。完整路径为：

```text
D:\Swansea-杜亦冯\biyeshj\output\tutorials\tiago_ik_beginner\.venv\Scripts\python.exe
```

右下角若显示 `No interpreter`，也可以从那里进入设置。已显示并使用正确解释器时，直接继续即可。

## 确认环境与安装依赖

在项目根目录的 PyCharm Terminal 中执行以下 PowerShell 命令：

```powershell
.\.venv\Scripts\python.exe --version
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -c "import numpy; print(numpy.__version__)"
```

预期版本为 Python 3.12.1 和 NumPy 2.5.4，依赖文件已固定为这次验证通过的 NumPy 版本。已经安装时显示 `Requirement already satisfied` 属于正常结果。

本教程通过 `requirements.txt` 安装依赖。将整个教程文件夹作为本地包执行 `pip install -e` 会因缺少打包配置而报错；关闭该提示，使用上面的命令安装即可。

## 运行与再次运行

在空白输出目录下，右键 `01_warmup_numpy.py`，选择 Run，默认结果位于 `runs/warmup_01`。你已完成这次运行，因此再次练习时使用新目录：

```powershell
.\.venv\Scripts\python.exe 01_warmup_numpy.py --out runs/warmup_02
```

继续使用 PyCharm Run 按钮时，在该脚本的运行配置中把 Parameters 设置为 `--out runs/warmup_02`，Working directory 设置为本教程项目文件夹。以后用 `warmup_03`、`warmup_04` 等新名字保存实验。

`ik_numpy.py` 是共用模块；练习入口是 `01_warmup_numpy.py`。这会生成二维两关节数据、训练一个有 160 个 ReLU 隐藏单元的网络，并计算末端误差。教学实现采用线性输出、Adam 和只从训练集拟合的 robust scaling。

## 对一个目标位置做预测

已新增 `03_predict_one.py`，它加载 `runs/warmup_01/model.npz`，预测目标位置 `(0.6, 0.4)` 米对应的两个关节角度，并用正运动学计算位置误差。

在 PyCharm Terminal 的 PowerShell 中运行：

```powershell
.\.venv\Scripts\python.exe 03_predict_one.py
```

也可以在编辑器中打开 `03_predict_one.py`，右键选择 Run。Python 的 `import`、`from` 等代码应写在 `.py` 文件中运行，或输入已经启动的 Python 控制台；不能直接作为 PowerShell 命令执行。

| 看到的提示符 | 当前环境 | 在这里输入什么 |
|---|---|---|
| `(.venv) PS D:\...>` | PowerShell，虚拟环境已激活 | 运行 Python 的命令，例如 `.\.venv\Scripts\python.exe 03_predict_one.py` |
| `>>>` | Python 交互控制台 | Python 代码，例如 `import numpy as np` |
| 编辑器中的 `.py` 文件 | Python 脚本 | 保存多行 Python 代码，然后右键 Run |

## 接入自己的静态 CSV

**本地项目已准备好 TIAGo 数据：`data/tiago.csv`。** 2026-10-15 从你下载的压缩包中提取了 `pykin/0_raw/ten_thousand_quaternion.csv`，共 10,000 条样本，原始内容未修改。文件来源与校验值见 `data/tiago.source.json`；它属于 TIAGo，后续 KUKA 实验仍需对应机器人的数据。

在本教程项目的 PowerShell 终端检查并开始训练：

```powershell
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv data/tiago.csv --check-only
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv data/tiago.csv --out runs/tiago_01
```

已实际验证检查结果为 `x_shape: [10000, 7]`、`y_shape: [10000, 7]`，表示位置加四元数输入与七关节输出。训练输出目录需要是新目录。

先用配套包内已有的二维 CSV 检查通用读取流程：

```powershell
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv runs/verified_warmup_data.csv --check-only
```

这条命令应输出 `x_shape: [6000, 2]`、`y_shape: [6000, 2]`，表示二维位置输入和两个关节输出。它验证的是 CSV 读取流程，不代表已经开始 TIAGo 实验。

若要练习从已有 CSV 训练，可运行下面的命令。输出目录需要是尚未存在的新目录：

```powershell
.\.venv\Scripts\python.exe 02_train_pose_csv.py --csv runs/verified_warmup_data.csv --out runs/csv_warmup_01
```

准备好真实 CSV 后，再将 `--csv` 改为真实文件路径。关节列应为 `arm_1`、`arm_2` 等，位置列为 `ee_x`、`ee_y`、`ee_z`；若带朝向，再提供 `ee_quat_x`、`ee_quat_y`、`ee_quat_z`、`ee_quat_w`。采用米、弧度和单位四元数，并保持坐标系、关节顺序与真实模型一致。

作者数据入口：https://doi.org/10.5281/zenodo.15365939 。若公开数据入口无法访问，也可按教程通过匹配的机器人模型生成数据。

CSV 训练器提供角度预测。对本次下载的 TIAGo Pykin 数据，已提供以下正运动学验证；其他机器人或数据需要核对相应模型。连续轨迹数据需要按轨迹或场景划分，不宜直接使用教学脚本中的独立样本随机划分。

## 检查 TIAGo 末端位置、朝向和关节限位

训练完成后，在 PowerShell 中运行：

```powershell
.\.venv\Scripts\python.exe 04_evaluate_tiago.py --run runs/tiago_01
```

这条命令读取已保存的模型和测试结果，无需重新训练或安装依赖。评估文件位于 `runs/tiago_01/fk_evaluation`：

- `结果解读.md`：中文指标表和解释，先看这个文件。
- `metrics.json`：完整数值、阈值、来源与校验信息。
- `per_sample.csv`：每条测试样本的位置和朝向误差，可用 Excel 查看。

重复运行只会更新评估目录。查看别的实验时，将 `--run` 改为相应目录，并在训练 CSV 不同的情况下同时传入 `--csv`。

评估模型是从作者引用目录的 URDF 提取、组合的运动学链，躯干升降固定为 0，基坐标为 `base_link`，末端使用 `arm_7_link`。它不是作者自行修改的 URDF 原文件。所有数值参数直接来自上游 URDF，已用这份 CSV 全部 10,000 条真实标签校验，重建误差接近浮点精度。模型与来源文件见 `models/README.md`。

**这份 Pykin CSV 的四元数列名虽然写作 `x,y,z,w`，实际数值顺序是 `w,x,y,z`。** 训练器将四列视为输入特征，因此本轮训练与测试保持一致，不需要仅为列名重新训练。评估脚本按已核对的实际顺序解释朝向；以后准备新目标位姿时，也必须按这个模型的实际输入顺序填写四元数。其他数据集需单独核对。

默认教学比较阈值为位置误差小于 10 mm、朝向误差小于 5 度。它们不是论文标准或工业要求。评估还检查关节限位，但没有碰撞或连续运动检查，因此阈值满足比例不等于完整任务成功率。后续方案使用验证集比较，锁定方案后再做最终测试。

## 层数、神经元数量和 ReLU

已核对你下载的 `tiago_dnn_ik-1.0.0.zip`，相关原文件复制在 `references/author_v1/tiago_dnn_mlp`。作者的 `simple_mlp_train.py` 只有一个隐藏 Dense 层，配置 `units: 160` 表示该层有 160 个神经元，不是 160 层。ReLU 逐个作用于隐藏神经元的加权和。作者 SimpleMlp 使用 tanh 输出；本教程采用线性输出，属于教学改写。

同一源码包另有 `deep_mlp_train.py`：配置 `num_layers: [20, 40, 20]` 和 `units: [128, 256, 128]`，实际构造 20 层 ×128、40 层 ×256、20 层 ×128，共 80 个隐藏层。这是另一个模型。源码和概念的详细说明见 `作者网络结构与ReLU说明.md`。

本教程的 CSV 训练器已支持指定隐藏层宽度，例如 `--hidden 160 160` 表示两个隐藏层，每层 160 个神经元。默认仍为一层 160。深层模型也可通过原有加载和 FK 评估流程读取。

容量对照脚本为 `05_compare_capacity.py`。本次验证实验保存在 `runs/tiago_capacity_01/验证集对照.md`：使用同一训练集与验证集比较一层 160、一层 320、两层各 160，以及朝向表示变化，没有对新方案进行测试集评估。两层网络是我们自己的对照，不是作者 80 层 DeepMlp 的复现。

`06_refine_tiago.py` 提供另一种方法：将已有网络输出作为初值，再用正运动学和数值迭代修正。默认评估验证集：

```powershell
.\.venv\Scripts\python.exe 06_refine_tiago.py --run runs/tiago_01 --split validation
```

本次结果在 `runs/tiago_01/ik_refinement/validation/结果解读.md`。它属于混合求解方法，不能将修正结果称为神经网络单独的精度。方案锁定后才使用 `--split test` 做最终测试；目前新增方案只比较了验证集。

## 已验证的练习

`runs/verified_warmup` 保存了一次已实际运行的二维教学结果，原始数据位于 `runs/verified_warmup_data.csv`。验证覆盖梯度、训练、模型保存和加载、数据集合隔离、训练集缩放和末端 FK 计算。

下载的 TIAGo 公开 CSV 已完成首轮训练，并通过运动学链、坐标系和四元数顺序核对。`04_evaluate_tiago.py` 已实际运行，可查看首轮测试集的末端位置、朝向与关节限位结果。尚未进行 KUKA 仿真或真实机器人验证。论文和原作者代码、数据的链接位于教程来源清单。

本教程是论文方法的教学改写。作者 Keras 和 Docker 路线依赖不同环境，若要复现原文数字，应单独配置，不能直接把其旧版依赖当成当前 Python 3.12 环境的依赖。
