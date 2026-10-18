# 从这里开始

适合你当前的 Windows、Python 3.12.1 和 PyCharm。完整说明见同目录的 `CoppeliaSim与PyCharm零基础指南.md`；PDF 放在 `output/pdf`。

2026-10-17：启动器已补上 CoppeliaSim 的安装工作目录。旧启动器从项目目录启动，可能造成界面或插件资源加载问题。若已用旧版打开，先保存需要保留的场景并关闭该窗口，再双击更新后的 `start_coppelia.cmd`。如果仍是黑屏，先把窗口缩小再最大化；再用 File > New Scene 检查默认场景是否也黑屏，以区分场景与显卡显示问题。

1. 如果 CoppeliaSim 还没打开，双击 `start_coppelia.cmd`，打开准备好的 KUKA 练习场景。已经开着时，按完整指南用 File > Open Scene 打开场景，避免多开。
2. 在 PyCharm 用 File > Open 打开本文件夹，选择 New Window，保留之前的 TIAGo 项目。
3. 解释器选择本文件夹的 `.venv\Scripts\python.exe`。这个环境已在本机准备好。
4. 下列命令均在本文件夹的 **PowerShell Terminal** 输入。先执行连接检查，成功后再继续。

```powershell
.\.venv\Scripts\python.exe 00_check_connection.py
.\.venv\Scripts\python.exe 01_select_frames.py
.\.venv\Scripts\python.exe 02_collect_static.py --samples 100
.\.venv\Scripts\python.exe 03_validate_csv.py --replay
```

运行 01 时需要两次切换到 CoppeliaSim：先单选左侧场景树的 `LBRiiwa14R820`，回终端按 Enter；再单选最后一个关节下面的 `link8_resp`，回终端按 Enter。始终保持仿真停止，不要点 Play。

03 默认检查第一次采集生成的 `data/kuka_static_100_01.csv`。再次采集会生成 `_02`、`_03` 等文件，需用 `--csv` 指定对应文件，详见完整指南。

这是七轴 KUKA 的静态运动学练习。`link8_resp` 的对象坐标系被选作练习末端，尚未标定真实工具 TCP，也未做碰撞筛选。回放一致性通过不代表神经网络已经学会逆运动学。

压缩包不包含虚拟环境；换电脑或换目录后应按完整指南重新建立 `.venv`，不要直接搬运原虚拟环境。

## 已有预测结果的 KUKA 位姿评价

`04_evaluate_predictions.py` 可以评价已有训练结果，默认读取 TIAGo 教学项目中 `runs/kuka_01` 的 KUKA 预测，以及本项目的 `data/kuka_static_10000_01.csv`。

在本 KUKA 项目的 PowerShell Terminal 中运行：

```powershell
.\.venv\Scripts\python.exe 04_evaluate_predictions.py
```

脚本会另开一个无窗口 CoppeliaSim 进程，不需要保留当前图形窗口，也不会操作已打开的场景。结果自动另存为训练结果目录中的 `coppelia_fk_test_01`、`_02` 等新文件夹。`fk_metrics.json` 的 `status: passed` 表示评价程序完成；模型性能要看位置和朝向误差，不能把它当成模型成功率。

评价其他训练结果时用 `--run`，其他数据用 `--csv`；这些相对路径都以本 KUKA 项目为起点。评价验证集预测时需加 `--split validation`，并保证对应训练记录也以验证集为评价集。后续模型比较应使用验证集，选定方案后再做独立测试。

## 逐点1mm目标的新计划

完整研究计划位于 `output/research/KUKA_1mm研究路线与实验计划.md`。本项目的 `configs/precision_1mm_plan.json` 保存目标和待实施的实验规则，状态为 `planned_not_executed`；它不是已执行的求解器配置，也不是已达到1mm的报告。

主线是先建立纯数值IK基线，再比较“神经网络初值＋有界数值修正”。第一阶段先完成逐点1mm验收器与七轴数值IK验证。独立KUKA环境已加入 NumPy2.5.4，供后续读取训练模型使用。
