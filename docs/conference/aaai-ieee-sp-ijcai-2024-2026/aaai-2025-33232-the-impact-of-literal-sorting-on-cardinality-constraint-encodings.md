---
title: The Impact of Literal Sorting on Cardinality Constraint Encodings
title_zh: 文字排序对基数约束编码的影响
authors: "Joseph E. Reeves, João Filipe, Min-Chien Hsu, Ruben Martins, Marijn J. H. Heule"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/33232/35387"
tags: ["query:sat-bva-cnf"]
score: 9.0
evidence: 文字排序在基数约束编码中生成辅助变量
tldr: 本文研究文字排序对基数约束CNF编码的影响，发现通过将相关文字相邻放置，编码会生成层级结构的辅助变量，使求解器能更抽象地推理。该方法优化了编码结构，与CNF预处理和辅助变量引入密切相关，为SAT公式压缩提供了新思路。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 现有基数约束编码优化聚焦于公式大小或传播强度，忽略了文字顺序。
method: 通过分析文字顺序对编码结构的影响，提出将相关文字相邻排列以生成层级辅助变量。
result: 实验表明该方法改善了求解器性能，优于传统指标。
conclusion: 文字排序是CNF编码优化的重要手段，与BVA中的辅助变量引入有共通之处。
---

## Abstract
The effectiveness of satisfiability solvers strongly depends on the quality of the encoding of a given problem into conjunctive normal form. Cardinality constraints are prevalent in numerous problems, prompting the development and study of various types of encoding. We present a novel approach to optimizing cardinality constraint encodings by exploring the impact of literal orderings within the constraints. By strategically placing related literals nearby each other, the encoding generates auxiliary variables in a hierarchical structure, enabling the solver to reason more abstractly about groups of related literals. Unlike conventional metrics such as formula size or propagation strength, our method leverages structural properties of the formula to redefine the roles of auxiliary variables to enhance the solver's learning capabilities. The experimental evaluation on benchmarks from the maximum satisfiability competition demonstrates that literal orderings can be more influential than the choice of the encoding type. Our literal ordering technique improves solver performance across various encoding techniques, underscoring the robustness of our approach.

---

## 论文详细总结（自动生成）

## 1. 核心问题与整体含义（研究动机和背景）

- 现代冲突驱动子句学习（CDCL）SAT 求解器的性能高度依赖于将问题编码为合取范式（CNF）的质量。基数约束（Cardinality Constraints）在众多问题（如 MaxSAT）中频繁出现，其编码质量直接影响求解效率。
- 已有工作主要聚焦于优化编码的**结构特征**：如公式大小（子句数和辅助变量数）、传播强度（弧一致性），但均忽略了基数约束内文字的顺序。
- 本文提出：通过**文字排序**（Literal Sorting）——即将相关文字在约束中相邻放置——可以改变编码中辅助变量的含义，使其形成层级抽象结构，从而帮助求解器针对相关文字组进行更有效的学习与推理。
- 核心洞察：文字排序对求解性能的影响有时甚至大于选择哪种编码类型本身。

## 2. 论文提出的方法论

### 核心思想
- 在不改变编码大小（子句数和辅助变量数）的前提下，通过重新排列基数约束中数据文字的顺序，改变编码生成的辅助变量所概括的变量集合，使辅助变量更好地反映问题的结构（如子句相关性、AMO 约束等）。
- 新的排序使得辅助变量能够更早、更廉价地传播单元，从而加速求解。

### 关键技术细节（排序方法，按复杂度递增）
- **Natural**：按变量名（整数标识）递增排序，反映问题生成时的自然顺序。
- **Random**：随机排列变量列表。用于对比无序化的影响。
- **Occur**：按变量在公式中出现的总次数（正负极性之和）降序排列，将高频变量放在前面。
- **Proximity**：基于子句结构的 BFS 式排序。具体流程：
  1. 初始化每个变量分为 0。
  2. 选择分数最高且未处理的变量（若最高分为 0，则选出现最多的变量），加入排序。
  3. 对该变量所在的每个 AMO 约束（若启用检测），未处理变量分数增加 len(K)²。
  4. 对该变量所在的每个子句 C，若长度 ≥3，则未处理变量分数增加 1/len(C)；若为二元子句，则分数增加 4。
  5. 重复直到所有基数约束中的变量处理完毕。
- **PAMO**：在 Proximity 基础上自动检测 AMO 约束（使用 BDD 工具 Guess&Verify，超时 50 秒）。
- **Graph**：构建变量共现图（变量为节点，共享子句则连边），使用 Louvain 社区检测算法运行最多 50 次（总超时 300 秒），选择社区数量最多且社区大小最均衡的划分，按社区顺序拼接变量。

### 组合策略
- **Natural+PAMO**：先 Natural 求解 100 秒，未解出则重启用 PAMO。
- **PAMO+Occur**：当公式子句数 <1e6 时用 PAMO，否则用 Occur。

### 编码类型
- 比较了五种常见基数约束编码：kmtotalizer、mtotalizer、cardinality network、sorting network、sequential counter。

## 3. 实验设计

### 数据集/Benchmark
- 来源于 2023 年 MaxSAT 竞赛无权重赛道（unweighted track）。
- 筛选条件：已知最优解，且最优界 >1 且 < 软子句数-1。得到 398 个可满足 + 398 个不可满足 SAT 实例（共 796 个）。
- 覆盖 30 个不同的 benchmark 家族（每个约 10 个实例），保证多样性。

### 对比方法
- 排序方法：Natural、Random（最好/最差 5 次）、Occur、Proximity、PAMO、Graph、Natural+PAMO、PAMO+Occur。
- 编码类型：kmtotalizer、mtotalizer、cardinality network、sorting network、sequential counter。
- 求解器：CaDiCaL（CDCL）。
- 评价指标：PAR-2 得分（平均运行时间，超时按 2 倍计）、求解数量（已解出实例数）。

## 4. 资源与算力

- 实验在 **StarExec** 平台上运行，每个任务配置 **32 GB 内存**，**1800 秒超时**。
- 未提及任何 GPU 信息；由于任务为 SAT 求解，通常仅使用 CPU。论文未说明 CPU 型号、数量或训练时长，因为非深度学习任务。

## 5. 实验数量与充分性

- **实验规模**：共 796 个实例，5 种编码 × 至少 6 种排序方法（加组合策略），形成密集实验矩阵（例如 Table 1 展示 5×5 结果，Table 2 展示 kmtotalizer 下的 8 种排序结果）。另有预处理时间曲线（图5）、累积求解曲线（图6）、子句覆盖率分析（图7）。
- **公平性**：使用标准竞赛 benchmark，对比了相同编码下的不同排序，以及不同编码下的最佳排序。结果报告 PAR-2 和求解计数，包含可满足与不可满足实例。
- **充分性**：实验覆盖了主要编码类型和多种排序策略，并对性能差异给出了结构性解释（子句覆盖率）。但仅局限于 MaxSAT 领域，未在其他应用领域（如规划、硬件验证）验证，可能存在领域偏倚。
- **消融实验**：通过对比 Natural 与各种排序，以及不同组合（如 PAMO+Occur）间接表明了各模块贡献。Random 的对比揭示了结构信息的重要性。

## 6. 论文的主要结论与发现

- **文字排序比编码类型选择更重要**：例如，使用 PAMO+Occur 排序的 mtotalizer、cardinality network、sorting network 的求解数量超过了 Natural 排序下的最佳编码 kmtotalizer。
- **Proximity 系列方法普遍有效**：PAMO+Occur 在所有编码类型上均显著提升性能，且 Par-2 较低（考虑预处理开销后仍优于 Natural）。
- **简单出现频率排序（Occur）效果不佳**：几乎与随机排序相当，说明仅靠频率无法捕获结构。
- **Graph 方法因预处理开销大，Par-2 较高**，但仍能解出更多实例，适合大型问题。
- **排序效果在可满足与不可满足实例上同步提升**（Table 2）。
- **良好排序的关键在于均衡的子句覆盖率**（图7）：避免辅助变量在前半部分过于粗粒度、后半部分过于细粒度。

## 7. 优点

- **视角新颖**：首次系统研究文字顺序对基数约束编码的影响，而非传统的大小或传播强度。
- **通用性强**：方法独立于编码类型，可即插即用于所有常见基数约束编码，且不改变公式大小。
- **实验设计全面**：对比多个排序方法和编码组合，使用标准竞赛 benchmark，评价指标客观。
- **开源的代码**（GitHub 链接），可复现。
- **提供解释性分析**：通过子句覆盖率揭示排序如何影响学习能力，深化理解。

## 8. 不足与局限

- **领域局限**：仅使用 MaxSAT 2023 基准，未在其他 SAT 应用（如规划、验证、密码学）上验证，推广性有待证明。
- **预处理开销大**：Proximity 和 Graph 方法在大型公式上可能超时（如超过 300 秒），部分策略（Natural+PAMO）虽缓解但仍非治本。
- **非通用最优排序**：VBS（虚拟最优求解器）远优于任何单一方法，表明需要实例自适应的排序选择，论文未提供自适应方案。
- **对某些编码改进有限**：如 sequential counter 对排序不敏感，文中未深入解释原因。
- **未与其他预处理技术（如变量消除、再编码）结合比较**，缺少更综合的评估。
- **Graph 方法随机性**：Louvain 算法多次运行可降低方差，但仍可能不稳定，且 50 次运行/300 秒超时可能不足以找到最佳社区。

（完）
