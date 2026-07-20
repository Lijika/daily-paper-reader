---
title: Towards Real-Time Approximate Counting
title_zh: 迈向实时近似计数
authors: "Yash Pote, Kuldeep S. Meel, Jiong Yang"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/33231/35386"
tags: ["query:sat-bva-cnf"]
score: 4.0
evidence: 减少近似计数中的SAT求解器调用次数
tldr: 模型计数在许多应用中需要大量SAT求解器调用，限制了其效率。本文提出ApproxMC7近似计数方案，相比现有方法减少了14倍的SAT调用次数，同时保持相同的理论保证。该算法通过优化采样和缓存技术，显著降低了计数查询的求解时间，适用于对时间敏感的计数应用。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 现有近似计数方法需要数百次SAT求解器调用，难以满足需要大量计数查询的实时应用。
method: 提出ApproxMC7方案，通过改进的哈希函数和更少的SAT求解器调用来实现高效近似计数。
result: ApproxMC7相比ApproxMC减少了14倍的SAT求解器调用，同时提供相同的概率保证。
conclusion: 该工作使得近似计数能够在低时间限制下高效运行，拓宽了模型计数在实时场景中的应用。
---

## Abstract
Model counting is the task of counting the number of satisfying assignments of a Boolean formula. Since counting is intractable in general, most applications use (ε, δ)-approximations, where the output is within a (1+ε)-factor of the count with probability at least 1-δ. 
Many demanding applications make thousands of counting queries, and the state-of-the-art approximate counter, ApproxMC, makes hundreds of calls to SAT solvers to answer a single approximate counting query.
The sheer number of SAT calls, poses a significant challenge to the existing approaches. 

In this work, we propose an approximation scheme, ApproxMC7, that is tailored to such demanding applications with low time limits. Compared to ApproxMC, ApproxMC7 makes 14× fewer SAT calls while providing the same guarantees as ApproxMC in the constant-factor regime. In an evaluation over 2,247 instances, ApproxMC7 solved 271 more and achieved a 2× speedup against ApproxMC.

---

## 论文详细总结（自动生成）

# 中文详细总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：模型计数（model counting）是计算布尔公式满足赋值个数的问题。由于精确求解是#P完全的，实际应用通常采用(ε,δ)-近似计数，即输出在(1+ε)因子内且概率至少1-δ。然而，现有最先进的近似计数器ApproxMC（及改进版ApproxMC6）为解决一次查询需要调用数百次SAT求解器（如ε=1/3时需336次SAT调用），这严重限制了在需要大量计数查询的实时应用中的效率。
- **研究动机**：许多应用场景（如自动化选择密文攻击、神经网络验证等）需要快速获取粗略估计，对时间预算敏感。现有方法因SAT调用过多而无法满足实时性要求。因此，亟需一种能在低时间限制下显著减少SAT调用次数，同时保持相同理论保证的近似计数方案。

## 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：将哈希分割的阈值（thresh）设为1，即每次只要求选定细胞中最多有1个解（即SAT可满足性检查），从而将一次估计中的SAT调用次数从数百次降至对数级别。同时通过精心设计的参数α、β及中位数技巧保证(ε,δ)-近似保证。
- **关键技术细节**：
  - **ApproxMC7总体框架**（Algorithm 2）：先检查公式是否不可满足；然后通过ComputeIter（Algorithm 3）计算所需迭代次数t（基于ε和δ）；每次迭代调用ApproxMC7Core获取一个估计，最终取中位数作为最终结果。
  - **ApproxMC7Core子程序**（Algorithm 4）：
    - 随机采样2-通用哈希函数h和向量λ。
    - 使用**galloping search**（倍增+二分）寻找最大的m，使得选定细胞（由h(m)和λ(m)确定）仍有解（即SAT）。过程中只调用一次SAT求解器判断是否可满足。
    - 返回估计值：√(2αβ) × 2^loIndex，其中α=β-1，β=(1+√(1+2(1+ε)^2))/2，满足2αβ=(1+ε)^2且α>1, β>2。
  - **正确性证明**（Lemma 5 & Theorem 1）：
    - 下界错误概率用马尔可夫不等式，上界用坎泰利不等式，避免了复杂的分支分析，证明仅一页。
    - 通过中位数技巧将单次估计的成功概率提升至1-δ，迭代次数t为O(log(1/δ)/(ε-1)^2)。
  - **SAT调用次数的理论分析**：ApproxMC7Core每次调用做log n次SAT求解（galloping search），总调用次数为O(log n * log(1/δ)/(ε-1)^2)。相比ApproxMC的O(thresh * log n * log(1/δ))，当thresh=Θ(1/ε^2)时，减少约14倍（典型ε=1/3）。

## 3. 实验设计
- **数据集/场景**：共2,247个实例，来自多个应用领域：
  - 神经网络定量验证（Baluta等人2019）
  - 模型计数竞赛2020-2022（Fichte等，Hecher等）
  - 程序合成（Alur等2013）
  - 定量控制即兴创作（Gittis等2022）
  - 软件属性量化（Teuber和Weigl 2021）
  - 自适应选择密文攻击（Beck等2020）
  - 同时包含模型计数和投影模型计数实例。
- **基准方法**：对比ApproxMC6（Yang和Meel 2023，即最新ApproxMC版本）。还使用精确计数器Ganak（Sharma等2019）获得精确计数以验证近似质量。
- **实验设置**：
  - 每个作业单核心运行，时间限制100秒，内存限制4GB。
  - δ=0.2，ε=1/3（即允许因子14的近似误差）。
  - 使用预处理器Arjun（Soos和Meel 2022）简化公式（神经网络基准除外，因其预处理器引入显著减速）。

## 4. 资源与算力
- 文中未提及GPU，全部实验在CPU上完成。
- 计算集群：AMD EPYC-Milan处理器，每个节点2×64核，512 GB RAM。每个作业使用单核。
- 未报告总训练时长或具体能耗，仅给出每个实例的运行时间（秒级）。

## 5. 实验数量与充分性
- **整体性能对比**：在全部2,247个实例上比较了求解数量、PAR-2得分和加速比。
- **近似质量验证**：在698个ApproxMC7和Ganak都能解出的实例上，将估计值与精确计数比较，观察是否落入[|sol|/14, 14|sol|]区间，并计算几何平均误差。
- **案例研究**：在自动化选择密文攻击场景中，使用101个CNF公式比较累计运行时间和加速比。
- **公平性**：对比方法为最新ApproxMC6，超时时间为100秒（与SAT竞赛惯例一致）；使用相同预处理器（神经网络除外）；统计指标包括PAR-2（惩罚平均运行时间），避免作弊。
- **充分性**：覆盖多个应用领域，包含不同难度实例；验证了近似保证和实际性能；案例研究体现了多查询场景下的优势。但未进行消融实验（如不同ε/δ设置下的性能变化，或不同哈希函数选择的影响）。

## 6. 论文的主要结论与发现
- **性能提升**：ApproxMC7比ApproxMC6多解决271个实例（1072 vs 801），PAR-2得分从139降至115（改善24），几何平均加速比2.1×，最大加速比达70×。
- **近似质量**：在698个验证实例中，仅6%的估计超出允许区间（小于理论保证的20%），几何平均观测误差为1.59（远小于ε=13）。
- **案例研究效果**：在自动化选择密文攻击中，ApproxMC7实现7×加速比，且所有实例均快于ApproxMC6。
- **SAT调用减少**：典型实例中ApproxMC7调用15次左右SAT求解器，而ApproxMC6需要数百次，减少约14倍（与理论吻合）。
- **证明简洁性**：ApproxMC7的正确性证明仅一页，远短于ApproxMC6的十五页，便于理解和扩展。

## 7. 优点
- **效率突破**：将每次估计的SAT调用次数从O(thresh)降至O(log n)，在低精度场景下实现数量级加速。
- **理论保证精确**：严格证明(ε,δ)-保证，并能适应ε>1的任意常数因子。
- **证明简洁**：利用马尔可夫和坎泰利不等式避免复杂分支，降低了分析难度和潜在错误。
- **实验覆盖全面**：使用大型多样基准，包含标准竞赛实例和实际应用，验证了性能和近似质量。
- **开源工具**：代码已开源（GitHub），便于复现和应用。

## 8. 不足与局限
- **适用范围受限**：论文明确针对ε>1的低精度场景（常数因子近似），未考虑高精度场景（如ε<0.5）。在需要高精度估计的应用中，ApproxMC7可能不适用或需要调整。
- **实验时间限制**：100秒超时，可能低估了长耗时实例上的性能差异；未测试更长时间限制下的表现。
- **近似质量验证**：仅对698个实例验证（部分实例因精确计数器无法求解而遗漏），且未检查不同ε/δ组合下的实际误差分布。
- **缺乏消融研究**：未分析galloping搜索、参数α/β选择、不同哈希函数族对性能的影响。
- **资源讨论不足**：未报告内存使用、可扩展性（如多核/分布式），也未提及对超大规模公式（变量数>10^6）的性能表现。
- **应用场景有限**：仅对一个实际应用（选择密文攻击）做了案例研究，其他场景未深入分析。

（完）
