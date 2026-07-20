---
title: Ordered Objectives in Maximum Satisfiability
title_zh: 最大可满足性问题中的有序目标
authors: "Jeremias Berg, André Schidler, Matti Järvisalo"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38429/42391"
tags: ["query:sat-bva-cnf"]
score: 6.0
evidence: 最大可满足性求解技术
tldr: 本文识别了最大可满足性问题中存在有序目标变量的实例类别，分析了问题结构对求解算法的影响，揭示了有序目标在常见基准测试中的普遍性，为改进MaxSAT求解器提供了新的视角。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 理解问题结构和编码如何影响MaxSAT求解算法的行为是一个重要挑战。
method: 通过分析实例中的约束条件对目标变量施加的顺序，定义有序目标类，并统计基准测试中该类实例的比例。
result: 发现大量常见MaxSAT基准实例具有有序目标，并识别了多个应用领域。
conclusion: 有序目标类为MaxSAT求解算法的设计与评估提供了新的视角。
---

## Abstract
Maximum satisfiability (MaxSAT) is a viable approach to solving NP-hard combinatorial optimization problems through propositional encodings. Understanding how problem structure and encodings impact the behaviour of different MaxSAT solving algorithms is an important challenge. In this work, we identify MaxSAT instances in which the constraints entail an ordering of the objective variables as an interesting instance class from the perspectives of problem structure and MaxSAT solving. From the problem structure perspective, we show that a non-negligible percentage of instances in commonly used MaxSAT benchmark sets have ordered objectives and further identify various examples of such problem domains to which MaxSAT solvers have been successfully applied. From the algorithmic perspective, we argue that MaxSAT instances with ordered objectives, provided an ordering, can be solved (at least) as efficiently with a very simplistic algorithmic approach as with modern core-based MaxSAT solving algorithms. We show empirically that state-of-the-art MaxSAT solvers suffer from overheads and are outperformed by the simplistic  approach on real-world optimization problems with ordered objectives.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

最大可满足性（MaxSAT）是解决 NP 难组合优化问题的一种重要声明式方法。尽管现代 MaxSAT 求解器（尤其是基于核的算法，如核心引导 OLL 和隐式命中集 IHS）取得了巨大成功，但**问题结构和编码方式如何影响求解器行为**仍是一个关键挑战。  
本文识别了一类特殊的 MaxSAT 实例——**目标变量存在顺序关系（ordered objectives）**，即约束条件强制要求目标变量之间满足蕴含链（例如 `b_k → b_{k-1}`）。这类实例在实践中广泛出现（如最小化最大值的 min-max 优化问题），但现有求解器并未利用这一隐藏结构，反而可能因此产生不必要的计算开销。  
作者旨在：① 定义并量化 ordered objectives 在标准基准中的普遍性；② 分析核心算法在该类实例上的理论行为；③ 提出简单的专用算法并与现有求解器进行实证对比。

## 2. 论文提出的方法论

### 核心思想
- **有序目标（ordered objective）**：存在目标变量排序 ≺，使得对任意解 α，若 b ≺ b′，则 α(b′ → b) = 1（即 b′ 为真时 b 必为真）。此时所有极小不可满足子式（MUS）均为单位子句，且最优解在目标变量上具有唯一模式。
- **几乎有序目标（almost-ordered objective）**：上述蕴含仅对**最优解**成立。两者通过“有序扩展”相互转换。

### 关键技术细节
1. **检测有序目标**：
   - 完全方法：对每对目标变量调用 SAT 求解器检查蕴含关系（二次开销）。
   - 近似方法：基于单元传播的 lookahead，快速获得蕴含子集，可判定有序性但可能漏判。
2. **有序目标下的最优解性质**：
   - 所有 MUS 均为单位子句（命题 1）。
   - 所有最优解在目标变量上赋值相同（推论 1）。
3. **核心算法退化分析**：
   - 若核提取总能返回最小核（MUS），则 CG-mus 和 IHS-mus 在有序实例上表现为**相同迭代序列**：每次迭代将下一个最小核对应的变量移除，迭代次数等于 MUS 个数（命题 4）。
   - 实际实现中，若提取非最小核，可能导致额外迭代和约束（例 5 展示了 IHS 需要 n+1 次迭代的极端情况）。
4. **简单算法 SimpleUS 和 SimpleSIS**：
   - **SimpleUS**（UNSAT/SAT 搜索）：按排序依次假设 `¬b_i`，若 SAT 则返回最优解；否则 `b_i` 必为 1。
   - **SimpleSIS**（解改进搜索）：从高到低试探，直到无法找到更低成本的解。
   - 两者均无需核提取或额外约束，仅利用有序性。

## 3. 实验设计

### 数据集与基准
- **基准检测**：MSE 2022–2024 全部实例（1558 个未加权，1545 个加权），去重后检测 ordered/almost-ordered 目标。
- **性能对比**：
  - **UW/W**：上述基准中检测出的有序实例（未加权 134 个，加权 29 个）。
  - **领域特定**：
    - **图着色（Col）**：415 个实例，源自 TreewidthLIB 和 DIMACS 图着色数据集。
    - **判断聚合（JA）**：50 个实例，基于 PrefLib 真实数据，编码 MaxHamming 规则。
    - **树宽（TW）**：287 个实例，来自 TreewidthLIB，限制图顶点数 ≤200。

### 对比方法
- **SimpleUS / SimpleSIS**：使用 PySAT + Cadical 1.9.5 的简单实现。
- **State-of-the-art 求解器**：
  - EvalMaxSAT（纯 OLL 变体 EvalNoSCIP 和先运行 SCIP 的 EvalSCIP）
  - MaxHS（IHS 算法）
  - Pacose（解改进型）
  - MaxCDCL（子句学习的分支定界）

### 实验环境
- 操作系统：Ubuntu 18.04
- 硬件：10 核 2.4 GHz Intel Xeon E5-2640 v4 CPU，160 GB 内存
- **每实例限制**：3600 秒时间，32 GB 内存（未使用 GPU）

## 4. 资源与算力

文中明确提及：
- 使用 **CPU 集群**，未使用 GPU。
- 每实例 **单核运行**（10 核物理机，但未提并行，通常每个作业使用单核）。
- 没有报告总的实验时长或 GPU 型号/数量，因此无法量化总体算力消耗。

## 5. 实验数量与充分性

| 实验类型 | 数据集 | 对比方法数 | 评价指标 |
|---------|--------|-----------|---------|
| 有序目标检测（表 1） | MSE 2022–2024 全部实例 | 无对比（仅检测） | 实例数统计 |
| 性能对比（表 2） | UW（134）、W（29）、Col（415）、JA（50）、TW（287） | 7 个方法（SimpleUS、SimpleSIS、EvalSCIP、EvalNoSCIP、MaxHS、MaxCDCL、Pacose）+ Virtual Best | #solved、PAR-2 |

- 共 **5 组主要对比实验**，覆盖未加权、加权及三个具体领域。
- 结果报告了 **求解个数和 PAR-2 惩罚平均运行时间**，可视化充分。
- **消融实验**：通过 EvalSCIP vs. EvalNoSCIP 对比了预求解的影响；通过 SimpleUS/SIS 对比了搜索方向。
- **公平性**：所有求解器使用相同时间/内存限制；简单算法使用相同 SAT 求解器（Cadical）作为 backend，与专业求解器中的 SAT 求解器可能不同，但已尽可能公平。
- **局限性**：
  - 仅测试了可被检测出有序目标的实例，未覆盖所有实际场景。
  - 加权实例中检测出的有序实例仅 29 个，统计意义可能受限。
  - 未进行交叉验证或多次重复实验（通常单次运行）。

## 6. 论文的主要结论与发现

1. **有序目标普遍存在**：约 10% 的 MSE 未加权实例和 2% 加权实例可检测为有序目标；约半数可判定实例具有几乎有序目标。
2. **核心算法对有序目标“退化”**：理论上，若核提取始终返回最小核，CG 和 IHS 的行为与简单线性搜索等价；但实际实现因非最小核、松弛变量、约束增加等导致额外开销。
3. **简单算法优于专业求解器**：在所有 5 个测试集上，SimpleUS 和 SimpleSIS 的求解个数与 PAR-2 均优于或等于所有 state-of-the-art 求解器（包括 EvalMaxSAT、MaxHS、Pacose、MaxCDCL）。Virtual Best 仅略优于 SimpleSIS，说明简单算法已接近最优。
4. **实践建议**：对于已知具有 ordered/almost-ordered 目标的 MaxSAT 实例，应采用基于有序性设计的简单搜索，避免复杂核心算法的开销。

## 7. 优点

- **理论分析严谨**：给出了有序目标的定义、MUS 唯一性证明、核心算法的退化分析（命题 4、例 5）。
- **实践洞察深刻**：不仅指出问题，还给出了可操作的简单算法，并用大规模实验验证。
- **检测方法实用**：提供了两种检测有序目标的方法（完全和近似），便于后续研究或工具集成。
- **覆盖多领域**：从抽象基准到具体应用（图着色、树宽、判断聚合），验证了普遍性。
- **代码和数据公开**：在 Zenodo 上提供补充材料（代码、详细结果、证明）。

## 8. 不足与局限

1. **有序目标检测不完整**：使用的 UP-based lookahead 只能检测部分有序实例，可能遗漏一些实际有序的实例，导致实验样本偏小。
2. **加权实例覆盖不足**：检测出的加权有序实例仅 29 个，且未单独展示领域加权实例的性能（除 UW/W 外其他领域均为未加权）。
3. **简单算法依赖排序**：SimpleUS/SIS 需要已知或能够推断变量排序，若排序未知则无法直接应用；虽然可通过检测得到，但增加了额外开销。
4. **没有探讨自动排序方法**：仅给出了检测算法，未提出如何高效地自动计算或利用部分有序性。
5. **实验仅限 CPU，没有 GPU 加速**：目前 MaxSAT 求解器多运行在 CPU 上，但未考虑大规模并行或异架构。
6. **随机性与统计显著性**：缺少多次重复实验的方差分析；一些实例个数较少（如 JA 仅 50 个），可能受个别实例影响。
7. **应用边界**：对于“几乎有序”的实例，简单算法（如 SimpleUS）不能直接保证最优性，需要先转化为有序扩展（增加约束）。该转换可能在部分场景中增加问题规模。

（完）
