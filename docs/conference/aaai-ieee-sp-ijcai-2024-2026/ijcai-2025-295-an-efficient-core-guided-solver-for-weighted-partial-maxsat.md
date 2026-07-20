---
title: An Efficient Core-Guided Solver for Weighted Partial MaxSAT
title_zh: 一种高效的加权部分MaxSAT核心引导求解器
authors: "(PDF |   Details)"
date: 2025-08-01
pdf: "https://www.ijcai.org/proceedings/2025/0295.pdf"
tags: ["query:sat-bva-cnf"]
score: 7.0
evidence: 提出了一种高效的核心引导加权部分MaxSAT求解器，推进了MaxSAT求解技术
tldr: 本文针对加权部分MaxSAT问题，提出了一种新的核心引导求解算法。该算法通过迭代识别并利用核心约束来引导搜索，显著提升了求解效率。在标准基准测试上，该求解器超越了现有最优算法。
source: IJCAI-2025-Accepted
selection_source: conference_retrieval
figures_json: "[{\"url\": \"assets/figures/ijcai-2025-accepted/ijcai-2025-295/fig-001.webp\", \"caption\": \"\", \"page\": 0, \"index\": 1, \"width\": 828, \"height\": 1572, \"label\": \"Figure\"}, {\"url\": \"assets/figures/ijcai-2025-accepted/ijcai-2025-295/fig-002.webp\", \"caption\": \"\", \"page\": 0, \"index\": 2, \"width\": 810, \"height\": 479, \"label\": \"Figure\"}, {\"url\": \"assets/figures/ijcai-2025-accepted/ijcai-2025-295/fig-003.webp\", \"caption\": \"\", \"page\": 0, \"index\": 3, \"width\": 813, \"height\": 477, \"label\": \"Figure\"}]"
tables_json: "[{\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-295/table-001.webp\", \"caption\": \"\", \"page\": 0, \"index\": 1, \"width\": 895, \"height\": 347, \"label\": \"Table\"}, {\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-295/table-002.webp\", \"caption\": \"\", \"page\": 0, \"index\": 2, \"width\": 876, \"height\": 283, \"label\": \"Table\"}, {\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-295/table-003.webp\", \"caption\": \"\", \"page\": 0, \"index\": 3, \"width\": 842, \"height\": 246, \"label\": \"Table\"}, {\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-295/table-004.webp\", \"caption\": \"\", \"page\": 0, \"index\": 4, \"width\": 1525, \"height\": 427, \"label\": \"Table\"}]"
motivation: 加权部分MaxSAT求解效率仍有提升空间，核心引导方法是一种有效范式。
method: 设计了一种新的核心提取和再利用策略，结合了基于核心的unsat核心分析和分支定界。
result: 在多个基准上取得了比现有求解器更快的求解时间和更好的求解质量。
conclusion: 核心引导方法在加权部分MaxSAT上具有显著优势。
---

## Abstract
No abstract is available.

---

## 论文详细总结（自动生成）

# 论文总结：An Efficient Core-Guided Solver for Weighted Partial MaxSAT

## 1. 核心问题与整体含义
- **研究动机**：最大可满足性问题（MaxSAT）是NP难的组合优化问题，加权部分MaxSAT（WPMS）是其重要变体，广泛应用于硬件验证、模型诊断、规划、数据分析等领域。现有核心引导（core-guided）求解器在效率上仍有提升空间，尤其面对权重差异大的实例时，传统分层策略可能导致求解时间过长。
- **整体含义**：论文旨在通过引入更智能的分层机制和快速提取多个不相交不可满足核的方法，显著提升WPMS求解器的最优解求解数和平均求解时间，推动MaxSAT求解技术的实际应用。

## 2. 方法论：核心思想与关键技术
- **核心思想**：基于核心引导的SAT求解框架，结合两种新策略：
  1. **扩展分层策略（Extended Stratification）**：
     - 目标：优先处理高权重软子句，避免在低权重子句上浪费算力。
     - 关键技术：根据软子句权重分布的离散程度（coefficient of variation < 1时跳过分层）动态决定是否启用分层；对于顶层，设置`assump_weight = max(avg_w, floor(max_w/2)+1)`，平衡细粒度与粗粒度。
     - 算法流程：根据SAT求解器返回状态（SAT/UNSAT/UNKNOWN）动态调整当前层的权重阈值，将权重低于阈值的子句移入`delay`集，逐个处理。
  2. **基于分层的多不相交不可满足核提取方法（MUCE）**：
     - 目标：在一次冲突分析中提取多个不相交的不可满足核，从而学习更多高质量子句。
     - 关键技术：先对每个核进行最小化（通过逐个移除测试得到MUS），然后利用当前分层阈值，对出现在核中的子句进行权重缩减，当剩余权重低于`assump_weight`时从临时假设集中移除，从而发现更多不相交核。
     - 算法流程：检查完所有核后，使用OLL过程将核转化为约束加入公式。
- **算法框架**（CASHWMaxSAT）：基于UWrMaxSAT，在每轮SAT求解后，若遇UNSAT则调用MUCE更新下界和公式；若遇SAT则更新上界并触发重新分层；同时集成了硬化策略、预处理、混合搜索、BMO等技术。

## 3. 实验设计
- **数据集与基准**：
  - 使用**MaxSAT Evaluation (MSE) 2022、2023、2024**完整加权赛道的**全部实例**，共计**1736个**（MSE22:607个，MSE23:558个，MSE24:571个）。
  - 实例涵盖组合优化、AI、电路设计、软件工程、生物信息学等多个领域。
- **对比方法**：
  - **独立求解器**（8个）：SCIP 8.1.0、Exact、Open-WBO、Pacose、WMaxCDCL、CGSS2、EvalMaxSAT、UwrMaxSAT（均为各年度最佳配置）。
  - **混合求解器**（5个）：WMaxCDCL+Open-WBO、MaxHS、EvalMaxSAT+SCIP、UwrMaxSAT+SCIP、WMaxCDCL+SCIP+MaxHS。
  - 自身变体（消融实验）：不启用分层（\STRA）、使用传统分层（+TSTR）、不启用MUCE（\MUCE）、不启用任何新策略（\MUST）。
- **评价指标**：求解成功的实例数（#solve）、平均求解时间（avg）、PAR2评分（惩罚平均运行时间，惩罚因子2）。

## 4. 资源与算力
- 论文**未明确说明使用的GPU型号、数量或训练时长**（本工作为CPU求解器，无需训练）。
- 实验硬件环境：
  - 操作系统：Ubuntu 22.04.5 LTS
  - CPU：Intel Xeon Platinum 8260 @ 2.40GHz（单机，未提及并行数量）
  - 内存：512GB
  - 每个实例时间限制：3600秒（与竞赛一致）。

## 5. 实验数量与充分性
- **实验数量**：
  - 独立求解器对比：在3个年度基准上共1736个实例，8个对比求解器。
  - 混合求解器对比：同样3个基准，5个对比混合求解器。
  - 消融实验：4种变体在3个基准上完整比较。
  - 额外分析：将时间延长至7200秒对前三名求解器重新测试MSE23。
- **充分性与公平性**：
  - 使用了公开标准基准，覆盖广泛领域。
  - 对比了当时最先进的SOTA求解器（包括独立和混合版本），配置为竞赛最佳版本或文中明确指定。
  - 消融实验逐一评估两个主要策略的贡献，并考虑了与现有分层策略（+TSTR）的对比。
  - 指标全面（#solve、avg time、PAR2），并提供了累计分布函数（CDF）图表。
  - 所有实验在相同硬件和时限下进行，保证了公平性。

## 6. 主要结论与发现
- **独立求解器对比**：CASHWMaxSAT在MSE22/23/24三个基准上均**解决最多实例**（423/431/438），且PAR2显著低于其他求解器（少至少100分以上）。平均求解时间也普遍更低。
- **混合求解器对比**：CASHWMaxSAT+SCIP在MSE23和24上超过其他所有混合求解器；在MSE22上略低于WMaxCDCL+SCIP+MaxHS（443 vs 444），但延长时限后反超。
- **消融实验**：
  - 关闭所有新策略（\MUST）性能最差，证明额外技术单独不够。
  - 引入扩展分层（\STRA）或MUCE（\MUCE）分别带来约5-12个实例提升。
  - 两种策略结合（完整算法）进一步带来约17个实例提升，表明它们协同作用强。
  - 传统分层（+TSTR）反而不如不用分层（\STRA），说明新分层策略与MUCE更好地整合。
- **核心结论**：设计的扩展分层策略和MUCE方法显著提升了求解效率和解决能力，在竞赛标准下取得了领先性能。

## 7. 优点
- **方法创新**：提出了自适应分层阈值（基于权重分布离散度）和结合分层的多核提取方法，既有理论直觉又有实际效果。
- **实验设计严谨**：在多个年度、大量实例上与众多SOTA求解器全面对比，并进行了系统的消融分析，验证了每个组件的贡献。
- **性能优越**：在独立和混合设置下均达到最佳或接近最佳的求解数，且PAR2最优。
- **可复现性**：论文中详细描述了算法伪代码和关键步骤，公开提交至竞赛，结果可验证。

## 8. 不足与局限
- **理论与局限性**：
  - 缺乏对算法复杂度的理论分析（如核提取的 worst-case 性能）。
  - 多核提取方法对`assump_weight`阈值敏感，在权重分布高度集中时可能无法有效提取不相交核。
- **实验覆盖**：
  - 仅测试了CPU环境，未讨论多核并行或分布式扩展。
  - 未在非竞赛类、极端大规模或实时性要求高的应用场景中验证。
  - 与其他混合求解器（如MaxHS）结合时，本文方法提升不如其他求解器明显（加SCIP后提升幅度较小），暗示在混合框架下需进一步优化SCIP接口。
- **潜在偏差**：
  - 对比的求解器版本为各年度竞赛最佳，但版本年份不完全一致（如SCIP 8.1.0为通用求解器，非专为MaxSAT优化）。
  - 部分对比求解器（如MaxHS）依赖商业求解器CPLEX，本文仅以CPU时间对比，未考虑许可或计算成本。
- **应用限制**：核心引导方法本身在实例几乎不可满足或所有软子句权重相等时可能效率下降；分层策略对权重分布敏感，不适用于非数值权重问题。

（完）
