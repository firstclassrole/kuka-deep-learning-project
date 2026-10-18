# 从这里开始

这是新建的 KUKA 逐点 1 mm 练习项目，适配本机 Python 3.12.1。数据、旧模型和场景已复制；旧项目保持不动。

先阅读 `KUKA逐点1mm新手教程.md` 或同目录 PDF。第一次只做环境与几何检查：

```powershell
Set-Location -LiteralPath 'D:\Swansea-杜亦冯\biyeshj\output\tutorials\kuka_1mm_beginner'
.\.venv\Scripts\python.exe 00_check_environment.py
.\.venv\Scripts\python.exe 01_check_geometry.py
```

然后运行数值基线和比较：

```powershell
.\.venv\Scripts\python.exe 02_run_experiment.py --mode numeric
.\.venv\Scripts\python.exe 04_inspect_result.py
.\.venv\Scripts\python.exe 02_run_experiment.py --mode compare
.\.venv\Scripts\python.exe 04_inspect_result.py
```

脚本自动启动、关闭独立后台 CoppeliaSim，不需要手动打开窗口、按 Play 或再次选择场景对象。结果默认自动存入新编号文件夹。

`numeric` 使用零位首个初值及训练集近邻/随机后备；`hybrid` 把首个初值换成已有神经网络预测，两者共用后备池。求解后用场景 FK 复核逐点三维误差。

**验收目标：全部目标的位置误差 ≤1 mm。**程序正常退出、平均误差很小都不能替代这个标准；还要报告朝向、关节限位及超时。

`03_make_audit.py` 只在开发结束、参数冻结后运行。当前配套文件没有提前生成最终审计目标。

本教程只涉及仿真选定末端，尚未做碰撞检测和实体工具 TCP 标定。

## 本机实测

环境检查通过；50 条标签 FK 重现通过。全部 2000 个开发验证目标实测如下：

| 方法 | 位置通过数 | 平均误差 mm | P95 mm | 最大误差 mm | 平均总耗时 ms |
|---|---|---|---|---|---|
| numeric | 2000/2000 | 0.056566 | 0.098153 | 0.977201 | 5.912 |
| hybrid | 2000/2000 | 0.057603 | 0.098385 | 0.996103 | 5.694 |

两者同时满足暂定 0.5° 朝向辅助阈值，均无超时、越限或无解。这是一次本机仿真测量，不足以证明混合方法有显著加速；更不等于整个工作空间或实体机械臂都已达标。

完整逐点结果保存在 `runs/compare_validation_008`，可用以下命令查看：

```powershell
.\.venv\Scripts\python.exe 04_inspect_result.py --run runs/compare_validation_008
```

`验证记录.json` 保存环境、参数、文件哈希和全部统计。最终新的 1000 个审计目标未生成、未运行。
