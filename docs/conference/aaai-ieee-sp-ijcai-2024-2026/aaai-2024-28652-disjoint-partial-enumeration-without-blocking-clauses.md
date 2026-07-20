---
title: Disjoint Partial Enumeration without Blocking Clauses
title_zh: 无阻塞子句的不相交部分枚举
authors: "Giuseppe Spallitta, Roberto Sebastiani, Armin Biere"
date: 2024-03-25
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/28652/29266"
tags: ["query:sat-bva-cnf"]
score: 7.0
evidence: 无需阻塞子句的SAT枚举新方法
tldr: 本文提出了一种无需添加阻塞子句即可枚举不相交部分模型的新方法。通过集成冲突驱动子句学习（CDCL）、时间回溯（CB）和模型收缩（Implicant Shrinking），解决了阻塞子句导致的内存消耗和传播减慢问题。实验证明了该方法的优势，它属于SAT求解技术的创新，但与BVA或CNF预处理无直接关系。
source: AAAI-2024-Accepted
selection_source: conference_retrieval
motivation: 传统使用阻塞子句的模型枚举方法会消耗大量内存并减慢单元传播，需要更高效的枚举方式。
method: 结合CDCL、时间回溯和模型收缩技术，在不引入阻塞子句的情况下枚举不相交部分模型。
result: 实验表明新方法在内存和速度上均优于传统方法。
conclusion: 该工作为SAT模型枚举提供了高效新途径，可推广到其他SAT应用。
---

## Abstract
A basic algorithm for enumerating disjoint propositional models (disjoint AllSAT) is based on adding blocking clauses incrementally, ruling out previously found models. On the one hand, blocking clauses have the potential to reduce the number of generated models exponentially, as they can handle partial models. On the other hand, the introduction of a large number of blocking clauses affects memory consumption and drastically slows down unit propagation. 
 We propose a new approach that allows for enumerating disjoint partial models with no need for blocking clauses by integrating: Conflict-Driven Clause-Learning (CDCL), Chronological Backtracking (CB), and methods for shrinking models (Implicant Shrinking). Experiments clearly show the benefits of our novel approach.

---

## 论文详细总结（自动生成）

# 中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **问题**：在命题逻辑AllSAT（求解所有满足性模型）中，现有方法主要分为两类：**阻塞子句（blocking clauses）法**和**非阻塞法**。阻塞子句法通过不断添加新子句排除已找到模型，虽然可以处理部分模型（partial models）从而压缩表示，但**大量阻塞子句导致内存膨胀和单元传播严重降速**；非阻塞法（如基于时间回溯CB）则只生成全赋值（total assignments），无法利用部分模型压缩，且容易陷入无解子区域。
- **动机**：需要一种既能枚举不相交（disjoint）部分模型（覆盖多个全模型），又能避免阻塞子句开销的方法，提高枚举效率。
- **整体含义**：本文提出无阻塞子句的不相交部分枚举框架，通过集成CDCL、时间回溯和模型收缩技术，在保持严格不相交的同时大幅降低内存和时间开销。

## 2. 论文提出的方法论

### 核心思想
- 利用**时间回溯（Chronological Backtracking, CB）**替代阻塞子句来确保模型不重复枚举（每个模型通过翻转最近决策变量获得下一个模型）；
- 使用**CDCL**在冲突时生成冲突子句进行分析，快速跳出无解子空间；
- 在找到全模型后，通过**模型收缩（Implicant Shrinking）**将全赋值转化为部分赋值，同时保证收缩后的部分模型仍满足公式且与其他部分模型不相交。

### 关键技术细节
- **搜索算法框架**：类似标准CDCL AllSAT，但冲突分析和模型返回后均执行**强制的时间回溯**（而非NCB的任意跳跃），并翻转当前决策变量。
- **冲突分析**：采用**last UIP**（唯一蕴涵点）分析，因为first UIP可能导致重复覆盖（Example 1给出反例）。
- **模型收缩算法（Chronological Implicant Shrinking）**：
  - 从当前总赋值尾部开始遍历，对每个决策变量，检查其是否可通过2-watched字面量机制被替换（即：该变量是否仍为当前子句满足所必须）。
  - **两种实现**：
    - **WatchList-check**：真正扫描子句，移除当前字面量并替换为另一个满足子句且仍在赋值中的字面量，更新watch list。
    - **Light-check**：仅检查同一子句中的另一个被监视字面量是否仍在赋值中（二元投影检查），更保守但更快。
  - 仅允许移除决策级别高于某一阈值的变量，保证收缩后仍与之前未访问的部分模型不冲突。
- **隐式回溯原因子句**：对于模型翻转产生的赋值，不显式存储其原因子句（该子句为所有更低决策级别决策变量的否定），而是引入`BACKTRUE`标记，在冲突分析时动态重构，避免存储膨胀。
- **决策变量排序**：采用VSADS启发式+非空watch list优先级+名称词典序。

## 3. 实验设计

- **数据集/benchmark**：
  - **Binary clauses**：合成问题，n个变量，二元子句形式，解数约\(3^{n/2}\)。
  - **Rnd3sat**：410个随机3-SAT问题，变量数10-50，clause/variable比例≈1.5（针对AllSAT调整）。
  - **CBS**（SATLIB）：1000个实例。
  - **BMS**（SATLIB）：500个实例。
  - 总计1960个实例，均具有高模型数且允许部分模型压缩。
- **对比方法**：
  - **BC**（Toda & Soh）——阻塞子句+总模型枚举。
  - **BC Partial**——BC的部分模型变体。
  - **NBC**（Toda & Soh）——非阻塞+总模型枚举。
  - **BDD**（Toda & Soh）——基于BDD的枚举。
  - **MathSAT5**（Cimatti et al.）——阻塞子句部分枚举。
- **评价指标**：超时1200秒内完整枚举所有模型的实例数；CPU时间对比；部分模型数量对比。

## 4. 资源与算力

- **硬件**：Intel Xeon Gold 6238R @ 2.20GHz，28核，128GB RAM，Ubuntu Linux 20.04。
- **未使用GPU或大规模预训练**：本文为SAT求解算法，无需GPU训练，仅CPU求解。
- 文中未提及训练时长或额外算力消耗，仅报告求解时间。

## 5. 实验数量与充分性

- **实验数量**：
  - 主表（Table 1）报告了4个benchmark、5种对比方法+本文方法的求解成功数，共计28组数据（4×7）。
  - 两类收缩算法对比（图1）：CPU时间散点图和部分模型数量散点图。
  - 与5种求解器的CPU时间散点图（图2）：每图包含所有1960实例。
- **充分性评价**：
  - **覆盖广泛**：包含合成、随机、SATLIB标准数据集，覆盖不同规模与难度。
  - **对比全面**：与主流AllSAT求解器（阻塞/非阻塞/BDD）比较，且包含部分模型变体。
  - **缺失消融实验**：未单独测试去除CDCL或CB的效果；未与最新非公开工具（如BASOLVER、ALLSATCC）比较。
  - **公平性**：超时一致，硬件相同，但未明确说明参数调优是否对所有工具公平（本文工具采用简化设置，未使用预处理、重启等）。

## 6. 论文的主要结论与发现

- **主要结论**：本文提出的**TABULAR ALLSAT**方法在大多数benchmark上**完成了最多实例的枚举**（总1939/1960，高于第二名BDD的1935），且CPU时间显著优于其他方法（除Rnd3sat上BDD略快）。
- **发现**：
  - 无阻塞子句+时间回溯+模型收缩的组合有效避免了传统阻塞子句的性能退化。
  - Light-check收缩算法在速度上优于WatchList-check（图1a），尽管压缩率略低（图1b）。
  - 在结构简单（低clause/variable比例）的随机问题上，BDD编译效率高，性能略优；但问题复杂度升高时（CBS、BMS），本文方法优势明显。

## 7. 优点

- **创新性**：首次将CDCL、时间回溯和模型收缩三者结合用于无阻塞子句的不相交部分枚举，理论扎实（基于Möhle & Biere 2019b的演算）。
- **效率**：避免阻塞子句导致的内存爆炸和传播降速；隐式原因子句进一步节省内存和GC开销。
- **实用性**：两种收缩算法提供速度和压缩率的权衡，适应不同场景。
- **可复现**：代码和benchmark已开源。

## 8. 不足与局限

- **实验局限**：
  - 未与基于知识编译（如D4、c2d）的方法比较，这些方法可能通过一次编译获得极高压缩率。
  - 未进行**消融研究**（如单独移除CDCL或收缩），无法量化每个组件的贡献。
  - 仅在单一硬件平台测试，未报告多次运行方差。
- **算法限制**：
  - Light-check收缩算法保守，可能错过更好的压缩；WatchList-check在模型极多时更新开销大。
  - 决策启发式仍较简单（VSADS+watch list），未针对CB优化。
  - 不支持投影枚举（projected enumeration）和组件缓存（component caching）。
  - 当前实现禁止重启和重相位（restarts/rephasing），可能限制在难解实例上的鲁棒性。

（完）
