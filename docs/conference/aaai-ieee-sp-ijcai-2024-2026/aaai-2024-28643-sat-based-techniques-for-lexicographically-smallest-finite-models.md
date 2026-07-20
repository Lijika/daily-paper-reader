---
title: SAT-Based Techniques for Lexicographically Smallest Finite Models
title_zh: 基于SAT的字典序最小有限模型计算技术
authors: "Mikoláš Janota, Choiwah Chow, João Araújo, Michael Codish, Petr Vojtěchovský"
date: 2024-03-25
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/28643/29248"
tags: ["query:sat-bva-cnf"]
score: 6.0
evidence: 基于SAT的技术
tldr: 本文提出基于SAT的技术来计算有限数学结构的字典序最小正规形，通过迭代构造和SAT求解器黑盒搜索，为代数结构的编目提供了自动化工具，展示了SAT在模型归约中的应用。
source: AAAI-2024-Accepted
selection_source: conference_retrieval
motivation: 为代数结构的编目提供一种自动化的正规形计算方法。
method: 将问题编码为SAT，逐步构造最小代表元并利用SAT求解器搜索。
result: 成功计算出多种有限结构的字典序最小表示。
conclusion: SAT求解器可有效用于计算数学结构的正规形。
---

## Abstract
This paper proposes SAT-based techniques to calculate a specific normal form of a given finite mathematical structure (model). The normal form is obtained by permuting the domain elements so that the representation of the structure is lexicographically smallest possible. Such a normal form is of interest to mathematicians as it enables easy cataloging of algebraic structures. In particular, two structures are isomorphic precisely when their normal forms are the same. This form is also natural to inspect as mathematicians have been using it routinely for many decades.

We develop a novel approach where a SAT solver is used in a black-box fashion to compute the smallest representative. The approach constructs the representative gradually and searches the space of possible isomorphisms, requiring a small number of variables. However, the approach may lead to a large number of SAT calls and therefore we devise propagation techniques to reduce this number. The paper focuses on finite structures with a single binary operation (encompassing groups, semigroups, etc.). However, the approach is generalizable to arbitrary finite structures. We provide an implementation of the proposed algorithm and evaluate it on a variety of algebraic structures.

---

## 论文详细总结（自动生成）

好的，遵照您的指示，以下是对该论文的详细中文总结。

### 论文总结：基于SAT的字典序最小有限模型计算技术

#### 1. 核心问题与研究动机

*   **核心问题**：如何为给定的有限数学结构（如群、半群等）自动计算其 **字典序最小（Lexicographically Smallest, LEXMIN）代表元**。这是一个数学结构的“正规形”。两个结构同构当且仅当它们的正规形相同。
*   **研究动机**：
    *   **数学家的需求**：字典序最小代表元是数学家编目代数结构的常用手段（自1955年起），它使研究者能够快速识别熟悉的结构（如将复杂的乘法表规约为直观的模运算表）。
    *   **统一交换格式**：在计算代数系统（如GAP）中，LEXMIN提供了一种在不同包之间交换数据的标准格式。
    *   **高效存储**：字典序的特性使得前缀树（trie）成为存储大量结构的高效方式，因为许多结构会共享LEXMIN的前缀。

#### 2. 方法论：核心思想与关键技术

论文提出了一种基于SAT求解器的逐步构造算法，而非直接对目标结构进行显式编码。核心思想是：

*   **核心思想**：逐步填充目标结构 `(D, ⋄)` 的乘法表，从左上角开始，逐行逐列地确定每个单元格的值。对于每个单元格 `(r, c)`，为每个可能的取值 `v` 生成一个SAT查询，询问是否存在一个同构映射 `f`，使得 `r ⋄ c = v` 成立。系统会从最小值1开始尝试，直到找到一个可满足的值，并将其固定。
*   **关键技术细节**：
    1.  **编码方式**：
        *   使用布尔变量 `xi→j` 表示一个未知的双射 `f`，其中 `f(i) = j`。并附加基数约束确保其为置换。
        *   约束 `r ⋄ c = v` 被编码为一系列子句：`(xi→r ∧ xj→c) ⇒ xi∗j→v`，其中 `i, j` 为输入域中的所有元素，`i ∗ j` 是已知的输入运算结果。
    2.  **逐步构造算法**：如算法1所示。算法通过嵌套循环，为每个单元格依次查询最小值，并逐步累积已知的赋值集合 `A`。
    3.  **效率改进（传播技术）**：为减少昂贵的SAT求解调用，提出了多种传播技术：
        *   **首行识别**：利用“幂等元”这一同构不变量。在输入表中，包含特定元素（如`e`）的行中，若`e*c=e`的列数最多，则此行必须映射到目标表的第一行。这可以减少解空间的搜索。
        *   **预算技术**（Budgeting）：基于观察：同构不会改变每行、每列或整个表中每个元素出现的次数（频数）。算法会统计输入表中的频数分布，作为“预算”。在尝试填充目标表时，如果一个元素在其所在行/列的“预算”已用完，则无需再为它发起SAT查询。
        *   **行不变量**：计算每行的某些不变量（如满足特定等式条件的列数、某元素的最小周期等）。当目标表的某行确定后，其不变量可与输入表的行进行匹配，从而唯一确定行映射关系，简化后续SAT查询。
        *   **上界剪枝**：每次成功的SAT查询都会找到一个完整的同构映射，由此可推断出目标表剩余部分的完整值。这些值可以作为后续单元格查询的上界，任何不小于它的值都无需再尝试。
        *   **搜索策略**：将线性搜索（逐个值尝试）替换为基于二分搜索的变体，以更快地找到每个单元格的最小可行值。

#### 3. 实验设计

*   **数据集与场景**：从五种代数结构中随机生成样本：**群 (Groups)、环 (Loops)、拟群 (Quasigroups)、半群 (Semigroups) 和一般代数 (Magmas)**。样本涵盖了从阶数16到128的210个实例，并额外包含了阶数为192和256的5个实例。
*   **Benchmark**：与未采用任何优化的“基本算法”进行对比。作者也试图与GAP包`Smallsemi`和一种显式编码方法对比，但前者因实现差异和时间/内存问题被排除，后者因内存消耗过大（数百GB）被排除。
*   **对比方法**：主要进行**消融研究 (Ablation Study)**，系统地对比了：
    *   **基本算法** (basic algorithm)
    *   **全增强版本** (all enhancements)
    *   依次关闭每个单项增强技术（如：无首行识别、无不变量、无预算、无中行预算、不同搜索策略等）的版本。
    *   此外，还对比了两种底层SAT求解器：**minisat** 和 **cadical**。

#### 4. 资源与算力

*   论文中明确指出了实验环境：一台配备 **Intel Xeon CPU E5-2630 v2 (2.6GHz) × 24核**，**64GB RAM** 的计算机。**未提及使用GPU**。
*   每个实例的计算超时时间设置为**30分钟**。

#### 5. 实验数量与充分性

*   **实验数量**：总共使用了 **210个随机样本**（来自5个结构种类 × 7个阶数 × 6个实例）加上额外的5个高阶实例，总计超过215个测试用例。进行了完整的消融研究，涵盖了所有关键改进技术的独立对比。
*   **充分性与客观性**：
    *   **充分**：测试覆盖了多种不同性质的代数结构（从高度约束的群到无约束的magma），且跨度较大（16到256阶），具有一定代表性。消融研究设计合理，能有效评估各技术的贡献。
    *   **公平**：所有对比的版本都是在同一套算法框架下进行的，唯一变量是启用的技术或使用的求解器，保证了对比的公平性。由于缺乏可直接对比的同类工具（显式编码和GAP包因故无法参与），实验的横向对比性有限。

#### 6. 主要结论与发现

*   **SAT技术可行**：基于SAT的逐步构造方法能够有效地为中等规模的有限代数结构计算字典序最小代表元。
*   **传播技术至关重要**：效率改进（特别是**预算技术**和**搜索策略**）对算法性能有决定性影响。没有它们，基本算法在面对许多实例时会超时。图2的消融研究清晰地展现了这一点。
*   **求解器选择影响不大**：在采用了所有优化技术后，使用 **minisat** 或 **cadical** 的性能表现非常接近（图3），远小于优化技术本身带来的差异。
*   **贡献总结**：本文的主要贡献在于提出了一种新颖的SAT算法框架，并设计了一系列有效的传播技术，将SAT求解器成功地应用于一个实际的计算代数问题。

#### 7. 优点

*   **方法新颖**：提出了一种“逐步构造”的范式，避免了对大型目标结构的显式编码，显著降低了基于SAT求解的计算复杂度（从`O(|D|^5)`空间复杂度降为`O(|D|^4)`）。
*   **技术实用**：设计的“预算”、“行不变量”等传播技术，有效规避了SAT求解器在处理计数约束（如鸽巢原理）时的弱点，显著提升了实际性能，体现了对问题特性的深刻理解。
*   **实用性强**：解决了计算代数领域一个具体的、有长期需求的实际问题，其输出形式（直观的乘法表）符合数学家的使用习惯，具有直接的应用价值。
*   **消融实验充分**：通过详尽的消融研究，定量地分析了每种改进技术的贡献，增强了结论的可信度。

#### 8. 不足与局限

*   **实验规模有限**：虽然测试了最高256阶的实例，但对于代数结构领域，更高阶数的结构（如500+）的研究同样重要。论文未测试更大规模的实例，算法的可伸缩性有待进一步验证。
*   **对比基准不足**：由于对比工具或因实现不同（Smallsemi），或因资源消耗过大（显式编码）而无法参与，实验的“SOTA”对比性不足。证明其优于其他方法的说服力有所欠缺。
*   **依赖SAT求解器性能**：算法性能高度依赖底层SAT求解器。虽然作者提到使用了增量式接口，但在面对极难的SAT实例（尤其是涉及强计数约束时）时，算法的鲁棒性仍有风险。
*   **通用性扩展**：虽然论文宣称方法可推广，但当前实现仅针对单一二元运算结构。扩展到多元运算或多关系结构，其编码复杂度和搜索空间会急剧增长，在工程实现和效率上可能面临严峻挑战。

（完）
