---
title: Structure-Aware Encodings of Argumentation Properties for Clique-width
title_zh: 结构感知的团宽度论证性质编码
authors: "Yasir Mahmood, Markus Hecher, Johanna Groven, Johannes K. Fichte"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39005/42967"
tags: ["query:sat-bva-cnf"]
score: 8.0
evidence: 紧凑的(Q)SAT编码用于求解
tldr: SAT求解器在小树宽实例上高效，但对于稠密图树宽可能很大。本文利用更一般的图参数团宽度提出结构感知的SAT编码方法，用于抽象论证推理。该编码相比传统树宽编码更加紧凑，能够有效处理稠密图结构的SAT实例，为理解编码局限性提供了新视角，并拓展了SAT求解器在稠密领域的应用。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有SAT编码多依赖树宽，但稠密图的树宽较大，限制了编码效率，因此需要更通用的图参数团宽度来设计紧凑编码。
method: 引入团宽度作为图结构参数，设计了结构感知的SAT编码方法，将抽象论证性质编码为紧凑的命题公式。
result: 该编码方案能够在团宽度较小的稠密图上产生紧凑的SAT实例，并且通过实验验证了编码的有效性。
conclusion: 工作表明团宽度是设计SAT紧凑编码的有力工具，为未来SAT预处理和公式压缩提供了新思路。
---

## Abstract
Structural measures of graphs, such as treewidth, are central tools in computational complexity resulting in efficient algorithms when exploiting the parameter. It is even known that modern SAT solvers work efficiently on instances of small treewidth. Since these solvers are widely applied, research interests in compact encodings into (Q)SAT for solving and to understand encoding limitations. Even more general is the graph parameter clique-width, which unlike treewidth can be small for dense graphs. Although algorithms are available for clique-width, little is known about encodings. 
We initiate the quest to understand encoding capabilities with clique-width by considering abstract argumentation, which is a robust framework for reasoning with conflicting arguments. It is based on directed graphs and asks for computationally challenging properties, making it a natural candidate to study computational properties. We design novel reductions from argumentation problems to (Q)SAT. Our reductions linearly preserve the clique-width, resulting in directed decomposition-guided (DDG) reductions. We establish novel results for all argumentation semantics, including counting. Notably, the overhead caused by our DDG reductions cannot be significantly improved under reasonable assumptions.

---

## 论文详细总结（自动生成）

### 论文总结

#### 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题**：图的结构参数（如树宽）已被广泛用于设计高效算法和紧凑的SAT编码，但树宽在稠密图上可能很大，限制了其适用性。团宽度（clique-width）是一种更一般的图参数，即使在稠密图上也能很小，但关于如何利用团宽度进行SAT编码的研究很少。
- **动机**：抽象论证（abstract argumentation）是知识表示领域的重要框架，其推理问题具有高计算复杂度（NP乃至Σₚ²）且天然基于有向图，适合作为研究团宽度编码能力的测试平台。
- **目标**：设计从论证问题到（Q）SAT的结构感知编码，使得编码后的公式的团宽度与原始论证框架的团宽度成线性关系，从而可以利用已知的团宽度求解算法（如计数SAT、QBF求解）高效地解决论证推理问题。

#### 2. 论文提出的方法论
- **核心思想**：利用论证框架的k-表达式（k-expression）作为指导，沿分解树逐节点构造等价SAT/QBF公式。这种编码称为**有向分解引导（DDG）缩减**，线性保持团宽度。
- **关键技术细节**：
  - 对每个语义（stable, admissible, complete, preferred, semi-stable, stage）设计特定的变量集合和编码公式。
  - 基础语义（第一层PH）：使用扩展变量、击败变量、攻击变量等，通过初始化、并集、重命名、边引入四种操作传递信息。
  - 最大化语义（第二层PH）：需要额外处理超集关系（preferred）或范围最大化（semi-stable, stage），通过构造QBF（∀∃）并利用引理13将混合范式的矩阵转化为纯DNF/CNF，保持团宽度线性。
  - 计数和推理（credulous/skeptical acceptance）通过追加单元子句（ea或¬ea）实现，正确性和团宽度保持自然继承。
- **关键定理**：每个语义对应的缩减正确性定理（双向一一对应）和团宽度线性保持定理（增加的额外颜色数为O(k)）。

#### 3. 实验设计
- **无实验**：本文为纯理论论文，未设计任何实验或数据集。所有结果均为数学证明和复杂度分析。

#### 4. 资源与算力
- 文中未提及任何计算资源（GPU、CPU集群等），因为方法不需要实际运行实验。

#### 5. 实验数量与充分性
- **不适用**：论文无实验，故无消融实验、对比实验等。理论证明的充分性通过严格的数学推理和复杂度下界保证。

#### 6. 论文的主要结论与发现
- 对于所有常见论证语义（stable, admissible, complete, preferred, semi-stable, stage），**存在DDG缩减**将问题转化为SAT或QBF，且编码后的团宽度与原论证框架的团宽度成线性关系。
- 利用已有的团宽度SAT/QBF算法（Fischer et al. 2008; Capelli & Mengel 2019），得到求解时间上界：
  - 第一层语义：\(2^{O(k)} \cdot \text{poly}(n)\)
  - 第二层语义：\(2^{2^{O(k)}} \cdot \text{poly}(n)\)
- 下界（基于ETH）：这些上界在指数部分不能显著改进。例如，对于可接受语义的credulous acceptance，不存在\(2^{o(k)}\)算法。
- 论证了有向团宽度比无向团宽度更适合本任务，因为无向信息会丢失攻击方向。

#### 7. 优点
- **理论创新**：首次将团宽度引入论证问题的SAT编码，并给出线性保持的构造，填补了团宽度编码的空白。
- **结构最优化**：证明编码的团宽度增加是线性因子（O(k)），且下界表明指数部分无法改进，表明编码在渐近意义下最优。
- **通用性**：覆盖所有主要论证语义，包括计数、credulous/skeptical推理，并且可用于第二层PH问题。
- **简洁性与可推广性**：编码基于k-表达式的递归操作，易于理解，且作者指出可以扩展到其他KRR形式（如abductive reasoning, ASP）。

#### 8. 不足与局限
- **无实验验证**：纯理论论文，未在任何实际SAT/QBF求解器上测试编码有效性，因此无法评估实际运行效率或与现有方法（如树宽编码）的比较。
- **实际可行性依赖**：需要预先知道论证框架的团宽度及其k-表达式，而计算最小团宽度是NP-hard的（但存在多项式时间近似算法）。
- **第二层语义的QBF编码**：虽然理论上可行，但矩阵需要转换为DNF，导致公式规模可能较大，实际求解可能仍困难。
- **应用范围**：仅针对抽象论证框架，未探讨其他领域（如规划、验证）的团宽度编码。

（完）
