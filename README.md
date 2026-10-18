# KUKA 机械臂毕业设计：完整项目文档归档

本归档保存从最初的文献阅读、研究规划、入门教程和实验结果说明，到 EG-353 项目 outline 的已生成文档。项目主题为使用深度学习预测 KUKA 机械臂的关节角度，并研究神经网络预测与数值逆运动学修正的组合方法。

归档包含 34 份项目文件：3 份 DOCX、6 份 PDF、1 份 XLSX、22 份 Markdown 和 2 份 TXT。文档保留原有 `output/` 相对路径，同一教程的导出副本和修订版按原位置保留；甘特图与风险评估工作簿统一放在 `output/spreadsheets/`。

## Excel 重制版

甘特图和风险评估已使用 Excel 重新排版。工作簿中的甘特图日期可编辑，Word 重制版的后四页采用 Excel 排版结果。旧原版 Word 保留，便于对照。新版 Word 已逐页检查，共 7 页；Excel 已通过原生重算和 4 页打印检查。

- [EG-353 项目 outline：Excel 重制版（Word）](output/documents/EG-353-POIP-2634068_Excel_Revision.docx)
- [甘特图与风险评估工作簿（Excel）](output/spreadsheets/KUKA_Project_Gantt_and_Risk_Assessment.xlsx)

## 按内容浏览

| 类型 | 目录入口 | 主要内容 |
| --- | --- | --- |
| 项目 outline | [documents](output/documents/) | 目标、简介、任务、里程碑、甘特图和风险评估 |
| 可编辑甘特图与风险评估 | [spreadsheets](output/spreadsheets/) | Excel 工作簿及用于 Word 重制版的排版内容 |
| 最初的资料阅读与文献检索 | [notes](output/notes/) | 阅读笔记的 Markdown、Word、TXT 版本及文献检索与初步规划 |
| 研究路线与方法核对 | [research](output/research/) | KUKA 1 mm 实验计划、综述低误差方法核对 |
| 教程 PDF 入口 | [pdf](output/pdf/) | CoppeliaSim 与 PyCharm 指南、KUKA 逐点 1 mm 教程 |
| KUKA 数据采集教程 | [kuka_data_collection](output/tutorials/kuka_data_collection/) | 环境入门、数据采集操作及教程 PDF |
| KUKA 逐点 1 mm 教程 | [kuka_1mm_beginner](output/tutorials/kuka_1mm_beginner/) | 教程说明、Markdown 和 PDF 版本 |
| 第一篇论文与 TIAGo 入门 | [tiago_ik_beginner](output/tutorials/tiago_ik_beginner/) | 论文新手教程、数据准备修订版、网络结构与 ReLU 说明、运行说明 |
| 实验结果与解读 | [runs](output/tutorials/tiago_ik_beginner/runs/) | 不同网络设置的实验说明、正运动学验证和数值修正结果解读 |

## 主要文档

- [EG-353 项目 outline](output/documents/EG-353-POIP-2634068.docx)
- [项目资料阅读笔记（Markdown）](output/notes/project_103642_reading_notes.md)
- [项目资料阅读笔记（Word）](output/notes/项目103642资料阅读笔记.docx)
- [项目资料阅读笔记（TXT）](output/notes/项目103642资料阅读笔记.txt)
- [文献检索与初步规划](output/notes/项目103642文献检索与初步规划.txt)
- [KUKA 1 mm 研究路线与实验计划](output/research/KUKA_1mm研究路线与实验计划.md)
- [综述低误差方法核对](output/research/综述低误差方法核对.md)
- [CoppeliaSim 与 PyCharm 零基础指南（PDF）](output/pdf/CoppeliaSim与PyCharm零基础指南.pdf)
- [KUKA 数据采集操作指南](output/tutorials/kuka_data_collection/KUKA数据采集操作指南.md)
- [KUKA 逐点 1 mm 新手教程（PDF）](output/pdf/KUKA逐点1mm新手教程.pdf)
- [第一篇论文新手教程（Markdown）](output/tutorials/tiago_ik_beginner/第一篇论文新手教程.md)
- [第一篇论文新手教程：数据准备修订版（PDF）](output/tutorials/tiago_ik_beginner/第一篇论文新手教程_数据准备修订版.pdf)
- [作者网络结构与 ReLU 说明](output/tutorials/tiago_ik_beginner/作者网络结构与ReLU说明.md)
- [运行说明](output/tutorials/tiago_ik_beginner/明天运行说明.md)

## 实验结果说明

| 实验或检查 | 结果文档 |
| --- | --- |
| KUKA 模型正运动学检查 | [结果解读](output/tutorials/tiago_ik_beginner/runs/kuka_01/coppelia_fk_test_01/结果解读.md) |
| TIAGo 初次训练的正运动学评估 | [结果解读](output/tutorials/tiago_ik_beginner/runs/tiago_01/fk_evaluation/结果解读.md) |
| TIAGo 数值修正验证 | [结果解读](output/tutorials/tiago_ik_beginner/runs/tiago_01/ik_refinement/validation/结果解读.md) |
| 10 层网络 | [实验结果](output/tutorials/tiago_ik_beginner/runs/tiago_10layers_01/实验结果.md) · [正运动学结果解读](output/tutorials/tiago_ik_beginner/runs/tiago_10layers_01/validation_fk/结果解读.md) |
| 2 层网络 | [实验结果](output/tutorials/tiago_ik_beginner/runs/tiago_2layers_01/实验结果.md) · [正运动学结果解读](output/tutorials/tiago_ik_beginner/runs/tiago_2layers_01/validation_fk/结果解读.md) |
| 作者 80 单元网络结构 | [实验结果](output/tutorials/tiago_ik_beginner/runs/tiago_author80_structure_01/实验结果.md) · [正运动学结果解读](output/tutorials/tiago_ik_beginner/runs/tiago_author80_structure_01/validation_fk/结果解读.md) |
| 网络容量比较 | [验证集对照](output/tutorials/tiago_ik_beginner/runs/tiago_capacity_01/验证集对照.md) |

## 阅读说明

- **1 mm 是研究目标，不是已经达到的实验结果。** 后续需要在固定、独立的测试集上验证位置误差、姿态误差、关节限位、失败情况及计算时间。历史实验说明中的结果仅适用于其记录的模型、数据与实验条件。
- Outline 中的 **2026/27 学年计划日期为拟定安排**，须按课程正式时间表及导师意见核对。
- 原版 outline 已完成内容和结构检查，**原版的渲染及分页效果尚未验证**，应在 Word 中检查后再正式提交。Excel 重制版 Word 已完成 7 页逐页检查，其工作簿已通过原生重算和 4 页打印检查。
- 归档仅收录已生成的阅读文档；原始私人邮件、教材与论文全文、上游源码、环境依赖、训练模型、CSV 数据、机器日志和 ZIP 包未收录。文档中引用的代码、数据或本机路径在此归档中可能无法打开。
- [manifest.json](manifest.json) 记录除其自身外每个实际文件的相对路径、字节数和 SHA256，用于检查文件完整性。

资料整理日期：2026-10-18。
