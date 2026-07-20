---
title: Parameterization of (Partial) Maximum Satisfiability above Matching in a Variable-Clause Graph
title_zh: 变量-子句图中匹配之上的(部分)最大可满足性问题参数化
authors: "Vasily Alferov, Ivan Bliznets, Kirill Brilliantov"
date: 2024-03-25
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/28628/29220"
tags: ["query:sat-bva-cnf"]
score: 7.0
evidence: SAT求解技术、参数化MaxSAT
tldr: "本文利用Gallai-Edmonds分解改进了变量-子句图中最大匹配参数化的MaxSAT上界，将运行时间从O*(4^k')降至O*(2.83^k')，并由此得到了(n,3)-MaxSAT和(n,4)-MaxSAT更优的指数时间上界，推动了MaxSAT算法的理论进展。"
source: AAAI-2024-Accepted
selection_source: conference_retrieval
motivation: 提高MaxSAT问题的参数化算法效率。
method: 使用Gallai-Edmonds分解改进上界，结合变量-子句图结构参数化。
result: 显著改进运行时间，并导出更优的指数时间算法。
conclusion: 证明了图论工具在MaxSAT算法设计中的有效性。
---

## Abstract
In the paper, we study the Maximum Satisfiability and the Partial Maximum Satisfiability problems. Using Gallai–Edmonds decomposition, we significantly improve the upper bound for the Maximum Satisfiability problem parameterized above maximum matching in the variable-clause graph. Our algorithm operates with a runtime of O*(2.83^k'), a substantial improvement compared to the previous approach requiring O*(4^k' ), where k' denotes the relevant parameter. Moreover, this result immediately implies O*(1.14977^m) and O*(1.27895^m) time algorithms for the (n, 3)-MaxSAT and (n, 4)-MaxSAT where m is the overall number of clauses. These upper bounds improve prior-known upper bounds equal to O*(1.1554^m) and O*(1.2872^m). We also adapt the algorithm so that it can handle instances of Partial Maximum Satisfiability without losing performance in some cases. Note that this is somewhat surprising, as the existence of even one hard clause can significantly increase the hardness of a problem.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **研究动机**：Maximum Satisfiability（MaxSAT）问题在人工智能、计算机科学和工程等领域具有重要理论意义和实际应用。尽管已有多年研究，但精确指数时间算法的上界仍有改进空间。本文聚焦于**参数化**视角，特别是“**高于变量-子句图中最大匹配**”的保证值参数化（即 \( \nu(F)+k \) 问题）。之前的最佳算法运行时间为 \( O^*(4^k) \)，本文希望显著降低该上界。
- **整体含义**：通过引入图论中的 **Gallai–Edmonds 分解**，作者将 \( (n-k) \)-Set Cover 问题的求解时间从 \( O^*(4^k) \) 降至 \( O^*(2^{\frac{3}{2}k}) \)（约 \( 2.83^k \)），从而直接改进 MaxSAT 参数化算法。此外，该结果还推导出对特殊限制（如每个变量出现次数不超过 3 或 4 次）的 MaxSAT 实例更优的指数时间上界，并首次系统处理了部分 MaxSAT（Partial MaxSAT）中硬子句的影响。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程
- **核心思想**：将 MaxSAT 问题归约到 \( (n-k) \)-Set Cover，然后利用 Gallai–Edmonds 分解分析集合覆盖中剩余元素图的结构，结合贪心策略和动态规划来降低搜索空间。
- **关键技术细节**：
    1. **参数约简与归约**：通过一系列归约规则（Reduction Rules）和分支规则（Branching Rules）将 Partial MaxSAT 实例逐步精简，最终转化为 \( (n-k) \)-Set Cover 实例。
    2. **Gallai–Edmonds 分解**：在 \( (n-k) \)-Set Cover 中，通过贪心选取覆盖至少 3 个新元素的集合，确保剩余元素形成的图中每个集合最多覆盖 2 个元素。对该图进行 Gallai–Edmonds 分解，得到三个集合 \( C, B, D \)，并利用其性质限制连通分量大小。
    3. **动态规划**：基于分解得到的集合 \( A' = A \cup B \)（A 是贪心覆盖的元素，B 是 Gallai–Edmonds 中的集合）和剩余顶点集合 \( V' \)，设计动态规划表 dp(X, Y)，按连通分量顺序逐步计算覆盖给定元素集所需的最少集合数。关键上界分析表明，连通分量大小不超过 \( 2(k - |A| + |R| - |B|) + 1 \)，结合组合计数得到总运行时间为 \( O^*(2^{\frac{3}{2}k}) \)。
- **算法流程（文字说明）**：
    - 输入：CNF 公式 F、整数 k。
    - 阶段 1：应用归约规则（Resolution、删除冗余子句、处理硬子句等）。
    - 阶段 2：分支规则处理（处理高出现次数的变量、包含两个文字的子句、有向图中的循环等）。
    - 阶段 3：等价转化为 \( (n-k) \)-Set Cover 实例。
    - 阶段 4：对 Set Cover 执行贪心覆盖（每次选覆盖至少 3 个新元素的集合），直到所有剩余集合只覆盖至多 2 个新元素。
    - 阶段 5：构造剩余图 G（顶点为未覆盖元素，边对应覆盖两个元素的集合），求最大匹配 M。
    - 阶段 6：若匹配足够大，直接输出 YES；否则执行 Gallai–Edmonds 分解，得到 B 和组件；按连通分量顺序执行动态规划，计算所需的最小集合数。
    - 输出：若所需集合数 ≤ n - k 则为 YES，否则 NO。

## 3. 实验设计：使用的数据集 / 场景、Benchmark、对比方法
- **数据集/场景**：作者使用**生成的特殊实例**，由一个三参数生成器（a, b, k）产生。该类实例满足 \( \nu(F) = ab \)，且最多可满足 \( ab + k \) 个子句。实例规模通过参数 b 线性增长（b 从 100 到 50000），a 固定为 20，k 固定为 10。这种设计旨在测试算法在参数 k 较小情况下的效率。
- **Benchmark**：无标准 MaxSAT Evaluation 数据集，因为该算法主要为参数 k 小的场景设计，而标准实例中参数通常很大，不适合直接比较。
- **对比方法**：对比了 11 个参与 MaxSAT Evaluations 2022 的公开求解器，包括：
    - cash-w-maxsat-coreplus, cgss, eval-max-sat, exact, max-cdcl, w-max-cdcl, max-hs, open-wbo, uwr-max-sat-scip, uwr-max-sat, w-max-cdcl-band-all。

## 4. 资源与算力
- **未明确说明**：论文未提及使用的 GPU 型号、数量或训练时长。实验是在普通 CPU 环境下进行，代码为 C++ 实现，运行时间以毫秒为单位测量。可以推测算力要求不高，与典型的基于分支的算法类似。

## 5. 实验数量与充分性
- **实验数量**：主要展示了一组实验结果（图 1），横坐标为 b（100 到 50000），纵坐标为运行时间（毫秒）。未报告多次运行的统计信息或标准偏差。
- **充分性与客观性**：
    - **积极方面**：对比了多个主流求解器，结果直观显示本文算法在测试实例上显著优于其他求解器（许多求解器在 b 较大时超过 30 秒超时）。
    - **不足**：
        - 仅使用一种生成模式，缺乏在真实世界 MaxSAT 实例上的评估。
        - 未设置消融实验（如验证每一步归约或分支规则的必要性）。
        - 未报告参数变化（如不同 a 或 k）下的性能，仅展示了 a=20, k=10 的情况。
        - 未进行统计显著性检验，图例中个别点的缺失仅说明超时，未提供具体的超时比例。
    - 总体而言，实验能证明算法在所设计场景下的优势，但公平性和全面性较弱。

## 6. 论文的主要结论与发现
1. **算法改进**：\( (n-k) \)-Set Cover 和 \( (\nu(F)+k) \)-MaxSAT 问题可在 \( O^*(2^{\frac{3}{2}k}) \) 时间内求解，将原有上界从 \( O^*(4^k) \) 大幅降低。
2. **部分 MaxSAT 推广**：证明当所有硬子句长度为 2 或更短时，Partial MaxSAT 也能达到相同上界；否则可在 \( O^*(2^{\frac{3}{2}k} + 2^k \cdot 1.12226^h) \) 时间内求解（h 为硬子句数）。
3. **应用结果**：导出 \( (n,3) \)-MaxSAT 和 \( (n,4) \)-MaxSAT 的新上界：\( O^*(1.14977^m) \)（优于 \( 1.1554^m \)）和 \( O^*(1.27895^m) \)（优于 \( 1.2872^m \)）。
4. **实验验证**：在生成实例上，本文算法远超所有对比的公开求解器，表明该算法在参数 k 较小时具有实用潜力。

## 7. 优点：方法或实验设计上的亮点
- **理论贡献显著**：利用 Gallai–Edmonds 分解这一强大的图论工具，成功突破了经典参数化算法瓶颈，上界改进幅度较大（\( 4^k \rightarrow 2.83^k \)）。
- **方法论系统性强**：归约规则、分支规则、动态规划环环相扣，逻辑严密，且所有规则均证明保持正确性。
- **对 Partial MaxSAT 的深刻洞察**：直观上硬子句会极大增加难度，然而作者发现特定情况下（硬子句长度 ≤2）仍可保持相同复杂度，这揭示了问题的结构特性。
- **实验对比全面**：涵盖了 11 个主流求解器，结果清晰展示了本文方法在目标场景下的压倒性优势。

## 8. 不足与局限：实验覆盖、偏差风险、应用限制
- **实验覆盖有限**：
    - 仅使用人工生成的特定模式实例，与真实应用中的 MaxSAT 实例（如电路设计、生物信息学等）差异较大。
    - 未在 MaxSAT Evaluation 标准基准集上测试，因此无法评估通用性。
    - 仅测试了一组参数（a=20, k=10, b 变化），缺乏更广泛的参数空间探索。
- **偏差风险**：
    - 生成器利用了已知的匹配结构，可能恰好适合本文算法的假设，导致对比结果偏向本文算法。
    - 对比求解器多为通用求解器，未针对这种参数化场景进行调优，可能不公平。
- **应用限制**：
    - 算法时间复杂度依赖于参数 k 较小（否则动态规划状态数会爆炸），对于参数大的实际实例效率极低。
    - 未实现常见的启发式技巧（如冲突驱动、核心引导），因此通用场景下竞争力不足。
    - 基于归约和分支的算法实现可能较复杂，且需要大量多项式时间操作，增加了常数因子。
- **缺少消融与统计分析**：未验证各个归约规则和分支规则的实际贡献，也未提供多次运行的方差信息。

（完）
