---
title: SAT-Based Tree Decomposition with Iterative Cascading Policy Selection
title_zh: 基于SAT的树分解与迭代级联策略选择
authors: "Hai Xia, Stefan Szeider"
date: 2024-03-25
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/28659/29280"
tags: ["query:sat-bva-cnf"]
score: 6.0
evidence: 通过局部改进缓解SAT翻译中的规模增长
tldr: 将问题翻译为SAT常导致公式规模剧增，限制其应用。本文针对树分解问题，提出基于SAT的迭代局部改进框架TW-SLIM，通过自动配置局部实例大小和SAT调用时间等参数，有效控制翻译后的公式规模。实验表明该方法能够在保证分解质量的同时显著减小SAT编码大小，为CNF公式压缩提供了实用方法。
source: AAAI-2024-Accepted
selection_source: conference_retrieval
motivation: SAT翻译方法会显著增加公式规模，限制了其在大型实例上的应用，需要优化翻译过程以减少冗余。
method: 提出TW-SLIM框架，利用多次局部SAT调用来逐步改进树分解的启发式解，并通过自动配置关键参数来平衡效率和质量。
result: 经过自动配置后，TW-SLIM能够生成更小的SAT编码，同时在树宽质量上保持竞争力。
conclusion: 该工作展示了通过迭代局部改进和参数自动配置来降低SAT编码规模的可行性，对CNF公式压缩有参考价值。
---

## Abstract
Solvers for propositional satisfiability (SAT) effectively tackle hard optimization problems. However, translating to SAT can cause a significant size increase, restricting its use to smaller instances. To mitigate this, frameworks using multiple local SAT calls for gradually improving a heuristic solution have been proposed. The performance of such algorithmic frameworks heavily relies on critical parameters, including the size of selected local instances and the time allocated per SAT call.

This paper examines the automated configuration of the treewidth SAT-based local improvement method (TW-SLIM) framework, which uses multiple SAT calls for computing tree decompositions of small width, a fundamental problem in combinatorial optimization. We explore various TW-SLIM configuration methods, including offline learning and real-time adjustments, significantly outperforming default settings in multi-SAT scenarios with changing problems.

Building upon insights gained from offline training and real-time configurations for TW-SLIM, we propose the iterative cascading policy—a novel hybrid technique that uniquely combines both. The iterative cascading policy employs a pool of 30 configurations obtained through clustering-based offline methods, deploying them in dynamic cascades across multiple rounds. In each round, the 30 configurations are tested according to the cascading ordering, and the best tree decomposition is retained for further improvement, with the option to adjust the following ordering of cascades. This iterative approach significantly enhances the performance of TW-SLIM beyond baseline results, even within varying global timeouts. This highlights the effectiveness of the proposed iterative cascading policy in enhancing the efficiency and efficacy of complex algorithmic frameworks like TW-SLIM.

---

## 论文详细总结（自动生成）

## 论文详细中文总结

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：将组合优化问题编码为SAT（可满足性问题）时，通常会导致公式尺寸大幅增大（立方或更差），从而限制SAT求解器仅能处理小规模实例。为克服这一瓶颈，研究者提出了基于多次局部SAT调用的框架（如SLIM——基于SAT的局部改进）。然而，这类框架的性能高度依赖于关键参数的选择（如局部实例大小、每次SAT调用的时间预算等），且由于实例在求解过程中动态变化，传统的静态配置方法难以适应。
- **整体含义**：本文以**树分解最小化（MinTW）**这一经典NP难问题为案例，系统研究了复杂SLIM框架——TW-SLIM（基于SAT的树分解局部改进方法）的自动配置问题，旨在提升框架在多种时间预算下的效率与效果，并为其他使用多次SAT调用的算法框架提供通用配置方法论。

### 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：结合离线学习与实时调整，提出**迭代级联策略（Iterative Cascading Policy, ICP）**，通过动态选择并轮换多个配置，在多次SAT调用过程中自适应地改进树分解。
- **关键技术细节**：
  - **TW-SLIM框架**：输入图G和初始启发式树分解T，反复选择子图（大小由局部预算lb限定），使用SAT求解器尝试降低子树的宽度，若成功则替换原分解，否则可进行“混洗”（shuffle，由参数sf控制）。关键参数包括：局部预算lb、局部超时lt、SAT调用超时st、混洗标志sf。
  - **离线配置**：使用SMAC对全训练集搜索最佳单配置（SB-all）；进一步**聚类基配置（CC）**：提取9个特征（7个与树分解相关，2个与图相关），用约束K-means将训练实例分为30个簇，每簇用SMAC独立搜索最优配置，得到30个配置池。通过比较，聚类方法优于单配置和AutoFolio。
  - **动态配置选择**：提出**CC-dyn**，在运行过程中根据更新后的特征实时选择簇配置，若当前配置无法进一步改善则随机尝试其他配置。
  - **迭代级联策略（ICP）**：
    - **静态版（ICP-sta）**：将30个配置按离线性能排序组成静态级联顺序，依次运行，每轮结束后保留最佳树分解，重复多轮直至全局超时。
    - **动态版（ICP-dyn）**：每轮根据当前实例特征，从30个不同级联顺序（对应30个簇）中选择最适合的级联顺序，实现自适应调整。
    - 变体：过滤配置（ub），去除从未在训练实例上取得唯一最佳性能的配置，减少级联长度。

### 3. 实验设计：数据集、基准、对比方法

- **数据集**：收集了来自多个公开源的大规模图实例（包括SDCC、PACE 2016/2017、UAI、Roadnet、TWlib），筛选后共**3331个实例**（顶点数100~10⁶，边数100~863,026），随机分为**训练集80%（2664个）和测试集20%（667个）**。
- **基准**：原始TW-SLIM的手动调参配置（lb=100, lt=1800, st=900, sf=1）作为基线。
- **对比方法**：
  - 离线方法：SB-all（全训练集单配置）、CC-sta（聚类静态选择）、CA（AutoFolio基于簇的算法选择）。
  - 在线方法：SB-one（每个实例上使用SMAC在线调优）、CC-dyn（聚类动态选择）。
  - ICP变体：ICP-sta（按TΣ或T#排序）、ICP-dyn（动态级联）、ICP-sta(ub)（过滤配置）。
- **评估指标**：总改进和TΣ（所有实例宽度减少量之和）和成功实例数T#。主要优化目标为TΣ。
- **超时设置**：全局超时从100秒到7800秒（默认），共5个时间点。

### 4. 资源与算力

- **集群硬件**：Linux (Ubuntu 18.04.6 LTS) Sun Grid Engine集群，共3个节点，每个节点配备**两颗AMD EPYC 7402 CPU**（每颗24核，2.80GHz），总计144个核心。
- **软件环境**：部分组件基于Python 2.7.5（TW-SLIM）、Python 3.5（AutoFolio）、Python 3.9.12（其他）。使用SMAC作为离线配置器（每任务限时48小时墙钟，4小时截止时间，300GB内存）。
- **未提及GPU**：实验全程仅使用CPU，未使用GPU加速。

### 5. 实验数量与充分性

- **实验规模**：在3331个实例上进行完整实验，分为训练和测试。对比了**9种方法变体**（手调、SB-all、CC-sta、CC-dyn、SB-one、ICP-sta×2指标、ICP-dyn×2指标、ICP-sta(ub)），并在**5种不同超时**（100s、500s、1000s、3000s、7800s）下重复测试，共进行了约**45组主要实验**。
- **消融与对比**：包括了离线 vs 在线、静态 vs 动态、不同排序指标、配置过滤等消融分析，实验设计较全面。
- **公平性**：所有方法在相同环境下运行，使用相同初始启发式（HTD v0.9.5-beta）和SAT求解器（Glucose），超时设置一致。但未提及对超时分配公平性的额外控制（如ICP因为多轮可能占用更多开销）。
- **充分性**：数据集覆盖多个领域，规模大，实验条件多样，结论可信。但仅针对一个具体问题（MinTW），泛化性需进一步验证。

### 6. 论文的主要结论与发现

- **离线单配置优于手调**：SB-all在测试集上TΣ提高59（从398到457）。
- **聚类基静态配置（CC-sta）进一步提升**：TΣ达500，优于SB-all和AutoFolio（CA）。
- **动态选择（CC-dyn）在长超时下优势明显**：7800秒时TΣ达619，比CC-sta提高24%，但短超时（≤500秒）反而下降。
- **迭代级联策略（ICP）全面超越**：ICP-sta(TΣ)在7800秒时达到TΣ=728，较手调提高83%，较SB-one（587）提高24%。在短超时（100秒）下仍保持优势（T#=168 vs 手调未给出，但高于其他方法）。
- **ICP-sta优于ICP-dyn**：在大多数超时下，静态级联（按TΣ排序）比动态级联表现更好，说明离线学习的级联顺序已足够有效，动态调整带来的开销可能得不偿失。
- **配置过滤（ub）在短超时下有利**：在100秒时略有提升，长超时下性能相近。

### 7. 优点

- **方法创新**：首次将离线聚类、动态选择和级联调度相结合，提出迭代级联策略，有效解决动态实例演化下的配置自适应问题。
- **实验广泛**：使用3000+大规模实例，涵盖多种实际应用，时间预算跨度大，对比充分，结果清晰。
- **实证结论深刻**：揭示在动态SLIM框架中，离线聚类配置优于单配置和传统算法选择器，而级联策略能显著提升性能；并指出静态级联在多数情况下优于动态级联，为后续研究提供参考。
- **可复现性**：提供了源代码和数据链接（Zenodo），便于复现和扩展。

### 8. 不足与局限

- **问题特定性**：所有实验仅针对MinTW问题，虽声称适用于其他SLIM框架，但未在其他问题上验证泛化性。
- **计算开销**：ICP需预先训练30个配置并运行多轮，训练阶段使用大量计算资源（SMAC 48h×30簇×8核），可能对资源有限的研究者不友好。
- **动态级联效果不佳**：ICP-dyn在大多数超时下不如ICP-sta，表明当前动态选择设计尚有改进空间（如特征更新频率、级联顺序质量）。
- **短超时表现有限**：所有方法在100秒超时下改进绝对值较低（TΣ<400），ICP虽领先但仍有限，说明短时间下SAT局部改进本身受限。
- **对比缺失**：未与纯启发式方法（如其他树宽下降算法）或不同SAT求解器进行比较，难以判断绝对效果。
- **未讨论SAT编码大小**：虽动机为减小SAT编码，但文中未直接测量编码规模，仅以树宽改进作为代理指标。

（完）
