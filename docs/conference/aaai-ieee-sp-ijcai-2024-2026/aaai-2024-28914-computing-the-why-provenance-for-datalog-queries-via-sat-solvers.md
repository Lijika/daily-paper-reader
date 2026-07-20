---
title: Computing the Why-Provenance for Datalog Queries via SAT Solvers
title_zh: 通过SAT求解器计算Datalog查询的why-provenance
authors: "Marco Calautti, Ester Livshits, Andreas Pieris, Markus Schneider"
date: 2024-03-25
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/28914/29739"
tags: ["query:sat-bva-cnf"]
score: 4.0
evidence: 利用SAT求解器计算Datalog查询的why-provenance
tldr: 本文展示了如何利用现成的SAT求解器高效计算Datalog查询的why-provenance。该方法将SAT求解技术应用于数据库溯源领域，但与BVA或CNF预处理无关。
source: AAAI-2024-Accepted
selection_source: conference_retrieval
motivation: Datalog查询的why-provenance计算开销大，需要高效方法。
method: 将why-provenance问题编码为SAT实例，使用现成求解器求解。
result: 比传统方法更高效地计算why-provenance。
conclusion: SAT求解器可有效用于数据库溯源任务。
---

## Abstract
Explaining an answer to a Datalog query is an essential task towards Explainable AI, especially nowadays where Datalog plays a critical role in the development of ontology-based applications. A well-established approach for explaining a query answer is the so-called why-provenance, which essentially collects all the subsets of the input database that can be used to obtain that answer via some derivation process, typically represented as a proof tree. It is well known, however, that computing the why-provenance for Datalog queries is computationally expensive, and thus, very few attempts can be found in the literature. The goal of this work is to demonstrate how off-the-shelf SAT solvers can be exploited towards an efficient computation of the why-provenance for Datalog queries. Interestingly, our SAT-based approach allows us to build the why-provenance in an incremental fashion, that is, one explanation at a time, which is much more useful in a practical context than the one-shot computation of the whole set of explanations as done by existing approaches.

---

## 论文详细总结（自动生成）

## 论文详细中文总结

### 1. 核心问题与整体含义（研究动机和背景）

- **问题**：Datalog查询结果的解释（why-provenance）是实现可解释AI的关键，但传统方法计算代价极高，尤其对于递归查询。why-provenance收集所有能推导出某个答案的输入数据库子集，但计算复杂度高，目前仅有少数实现。
- **动机**：需要一种高效、可增量计算的why-provenance方法，同时避免传统证明树中不合直觉的解释（如循环推导、歧义推导）。
- **背景**：Datalog广泛应用于本体查询回答，现有方法（如Elhalawati等2022）基于存在规则改写，但计算效率低且不支持增量输出。

### 2. 方法论：核心思想、关键技术细节与流程

- **核心思想**：利用现成的SAT求解器，通过将why-provenance问题编码为布尔可满足性问题，实现高效且增量式的解释生成。
- **关键技术细节**：
  - **引入无歧义证明树（Unambiguous Proof Trees）**：要求证明树中同一个事实的所有出现必须对应同构的子树，避免歧义推导，从而允许使用有向无环图（DAG）紧凑表示。
  - **压缩DAG**：无歧义证明树可压缩为根DAG，其叶子集合与证明树的支持集相同。
  - **布尔公式构造**：
    - 基于事实的向下闭包（downward closure）构建超图，表示所有可能的压缩DAG。
    - 公式变量分为三类：节点变量（表示事实是否在DAG中）、超边变量、边变量。
    - 公式包含四个子公式：`φgraph`（节点-边一致性）、`φroot`（根节点唯一）、`φproof`（每个内节点恰有一个规则实例）、`φacyclic`（无环约束，采用顶点消除编码）。
  - **增量枚举**：通过添加阻塞子句（blocking clause）重复调用SAT求解器，每次获得一个不同的解释，直到公式不可满足。
- **算法流程**（文字说明）：
  1. 对查询Q、数据库D和给定元组t，使用DLV引擎计算向下闭包。
  2. 基于向下闭包构造布尔公式。
  3. 调用SAT求解器（Glucose）求解，得到一个满足赋值，从中提取叶子事实集作为第一个解释。
  4. 添加阻塞子句排除当前解释，重复求解，直到无解。

### 3. 实验设计

- **数据集/场景**（共五个场景）：
  - **TClosure**：比特币网络（235K事实）、Facebook网络（88.2K事实），2条规则。
  - **Doctors**：7个不同配置的合成数据库（100K事实），6条规则。
  - **Galen**：4个数据库（26.5K～82K事实），14条规则。
  - **Andersen**：5个数据库（68K～6.8M事实），2条规则。
  - **CSDA**：httpd（10M）、postgresql（34.8M）、linux（44M）三个大型数据库，2条规则。
- **对照方法**：仅与Elhalawati等2022的基于存在规则的实现（规则基方法）在Galen和Doctors场景上比较（因其他场景缺乏改写规则）。
- **评测指标**：
  - 构建向下闭包和公式的时间（图1）。
  - 生成每个解释的延迟时间（图2）。
  - 端到端运行时间（图3，对比实验）。

### 4. 资源与算力

- **硬件**：Intel Core i7-10750H CPU @ 2.60GHz，32GB RAM，Fedora Linux 37。
- **软件**：Python 3.11.2，C++编译g++ 12.2.1（-O3优化），SAT求解器Glucose 4.2.1，Datalog引擎DLV 2.1.1和VLog 0.9.0。
- **未使用GPU**，所有实验在单台笔记本上完成，未提及训练过程（无深度学习）。

### 5. 实验数量与充分性

- **实验数量**：
  - 每个场景的每个数据库随机选取100个查询答案元组，共约5个场景×多个数据库 = 数百个元组。
  - 每个元组允许生成最多10,000个解释，超时5分钟。
  - 对比实验仅覆盖Galen和Doctors场景（共5个数据库），每个数据库对应约100个元组。
- **充分性评价**：
  - **充分**：覆盖了不同规模（从26K到44M事实）和不同规则复杂度（2～14条规则）的数据集，包含合成数据和真实数据。
  - **客观**：随机选取元组，报告统计分布（箱线图），避免单一结果偏差。
  - **公平**：对比方法使用其默认实现（VLog 0.9.0），但仅限于已有改写的场景，未覆盖所有场景；论文指出作者未提供全部改写工具。
  - **不足**：对比实验规模有限（仅两个场景），且缺乏与使用更先进Datalog引擎（如Soufflé）的why-provenance方法（如Zhao等2020）的直接对比。

### 6. 主要结论与发现

- SAT基方法**显著优于**规则基方法：在Galen场景中，规则基方法对41/100的元组超时（5分钟），而SAT方法全部在几秒内完成；在Doctors场景中，SAT方法也更快。
- **增量生成速度极快**：一旦公式构建完毕，每个解释的延迟大多低于1毫秒，中位数在微秒量级。
- 公式构建时间主要消耗在向下闭包计算上（使用DLV），对于最大数据库（CSDA）需要几十秒到数分钟，但仍可接受。
- 无歧义证明树的概念不仅提供了更直观的解释，还为SAT编码提供了便利（允许使用压缩DAG）。

### 7. 优点

- **创新性**：首次将SAT求解器用于why-provenance计算，且利用无歧义证明树实现紧凑编码。
- **实用性**：增量式生成解释，每次只产生一个解释，便于交互式探索。
- **效率**：相比现有方法，在大多数场景下运行时间低1～2个数量级。
- **成熟技术利用**：直接使用高效的C++实现和商用SAT求解器Glucose，无需额外训练。
- **可复现**：公开了代码和数据（GitLab链接）。

### 8. 不足与局限

- **实验覆盖**：
  - 对比实验只覆盖两个场景，缺少与更多基线（如基于BFS的证明树枚举或使用Boucles等工具）的对比。
  - 未测试极端递归查询（如多个互递归规则）或含否定规则的Datalog扩展。
- **构建向下闭包的瓶颈**：虽然SAT求解部分很快，但向下闭包构建时间占主导，对于超大规模数据库（>10M事实）可能需要更长时间。
- **无歧义性限制**：无歧义证明树排除了某些实际合理的推导（例如一个事实可由多种等价方式推导），可能丢失部分解释。
- **硬件限制**：实验在单台笔记本上进行，未测试分布式或集群环境；但论文声称该结果已表明方法高效。
- **未评估模型可解释性**：仅评估计算效率，未通过用户研究验证无歧义解释是否更符合用户预期。
- **依赖特定Datalog引擎**：向下闭包计算依赖于DLV，但可替换为其他引擎。

（完）
