---
title: Certified Branch-and-Bound MaxSAT Solving
title_zh: 认证的分支定界MaxSAT求解
authors: "Dieter Vandesande, Jordi Coll, Bart Bogaerts"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38449/42411"
tags: ["query:sat-bva-cnf"]
score: 7.0
evidence: 提出了分支定界MaxSAT求解的认证证明记录，属于先进的SAT求解技术
tldr: 本文针对MaxSAT求解器，提出了在分支定界框架下的证明记录方法，使得求解结果的正确性可以被形式化验证。该方法填补了MaxSAT领域证明记录的空白，提升了SAT求解器的可信度。实验表明该方法对性能影响较小。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: MaxSAT求解器可能存在软件错误，需要可验证的证明日志。
method: 在分支定界MaxSAT求解过程中添加证明日志生成和验证步骤。
result: 成功为多种MaxSAT技术生成证明，且运行时开销可接受。
conclusion: 为MaxSAT求解提供了必要的正确性保证。
---

## Abstract
Over the past few decades, combinatorial solvers have seen remarkable performance improvements, enabling their practical use in real-world applications.
In some of these applications, ensuring the correctness of the solver's output is critical.
However, the complexity of modern solvers makes them susceptible to bugs in their source code.
In the domain of satisfiability checking (SAT), this issue has been addressed through proof logging, where the solver generates a formal proof of the correctness of its answer.
For more expressive problems like MaxSAT, the optimization variant of SAT, proof logging had not seen a comparable breakthrough until recently. 

In this paper, we show how to achieve proof logging for state-of-the-art techniques in Branch-and-Bound MaxSAT solving. 
This includes certifying look-ahead methods used in such algorithms as well as advanced clausal encodings of pseudo-Boolean constraints based on so-called Multi-Valued Decision Diagrams (MDDs).
We implement these ideas in MaxCDCL, the dominant branch-and-bound solver, and experimentally demonstrate that proof logging is feasible with limited overhead, while proof checking remains a challenge.

---

## 论文详细总结（自动生成）

# 论文详细总结

## 1. 核心问题与整体含义（研究动机和背景）

- **问题背景**：组合求解器（如 SAT 求解器）在实际应用中日益普及，尤其在安全关键系统中，求解结果的正确性至关重要。然而复杂求解器易受代码漏洞影响，可能给出错误答案。SAT 领域已通过**证明日志（proof logging）** 来形式化验证求解器输出正确性，但 MaxSAT（SAT 的优化变体）上的证明记录直到最近才取得进展。
- **核心问题**：此前证明日志仅覆盖了 MaxSAT 的两种主流范式（基于解改进搜索和基于核心指导搜索），而**分支定界（Branch-and-Bound）** 范式尚未有认证方案。分支定界求解器（如 MaxCDCL）结合了冲突驱动子句学习（CDCL）和前瞻（look-ahead）下界估计，其核心挑战在于如何证明前瞻过程中产生的加权局部核及多值决策图（MDD）编码的正确性。
- **整体含义**：本文提出首个针对分支定界 MaxSAT 求解的完整证明日志方案，提升了 MaxSAT 求解的可信度，为安全关键应用提供了形式化保障。

## 2. 方法论：核心思想、关键技术细节

### 2.1 证明系统基础
- 采用 **VeriPB** 证明系统，支持切割平面推理（线性组合、除法、饱和等）、反向单元传播（RUP）以及冗余性规则（用于重言式引入和矛盾证明）。
- 优化目标：最小化目标函数 \( O = \sum v_i b_i \)，每次找到更优解时添加“解改进约束” \( O \le v^* - 1 \)。

### 2.2 分支定界求解框架的认证
- **MaxCDCL 算法**（Algorithm 1）：标准 CDCL 流程，但在每次单元传播后调用前瞻过程（Algorithm 2）判断当前节点是否可能改进最优解。若前瞻判断不可改进，则学习新子句并回溯。
- **前瞻过程的认证**：
  - 定义**加权局部核** \( \langle w, R, K \rangle \)：在给定部分赋值 \( \alpha \) 下，若假设 \( R \subseteq \alpha \) 且 \( K \) 中文字均为假，则产生矛盾（即 \( F \land R \land K \models \bot \)）。其中 \( K \) 仅含目标文字（objective literals）的否定，\( w \) 为权重。
  - 定义 **O-兼容性**：对每个目标文字 \( \ell \)，所有核中包含 \( \ell \) 的权重之和不超过 \( w_O(\ell) \)。
  - 利用 O-兼容核集可推导两个关键结论：
    1. **软冲突**：若总权重 \( w_O(Q) \ge v^* \)，则当前节点不可改进。
    2. **硬化（hardening）**：若对未赋值目标文字 \( \ell \) 有 \( res(\ell, Q) + w_O(Q) \ge v^* \)，则可安全地将 \( \ell \) 传播为非代价产生。
  - **证明生成**：
    - 每个局部核对应的子句 \( C_q = \bigvee_{\ell \in R \cup K} \ell \) 可通过 RUP 直接证明（因前瞻中的传播已导致矛盾）。
    - 基于核集 \( Q \) 和解改进约束，利用切割平面推导出学习子句。定理 5 给出了构造性推导过程：至多 \( 3|O|+2|Q|+1 \) 步。
    - 若冲突发生于根层，则通过 RUP 证明 \( 0 \ge 1 \)，结合目标改进规则证明最优性。

### 2.3 MDD/BDD 编码的认证
- **编码原理**：为解改进约束 \( O \le v^* - 1 \) 构建有序归约的 BDD 或 MDD（允许在存在互斥约束时分支到一组变量）。每个内部节点 \( \eta \) 引入新变量 \( v_\eta \)，并添加子句 \( b + \overline{v_{\eta_t}} + \overline{v_\eta} \ge 1 \) 和 \( \overline{v_{\eta_f}} + \overline{v_\eta} \ge 1 \)，以及顶层单元子句 \( v_{\eta_{\top}} \ge 1 \)。
- **认证挑战**：BDD 节点可能代表多个等价的伪布尔约束（如 \( \sum_{i\ge k} v_i b_i \le l \) 与 \( \le u \) 等价，当区间 \( [l, u] \) 内所有值对应相同布尔函数）。需证明这些等价性，而判断某具体值是否可达是 NP 难的。
- **解决方案**：
  - 对每个节点 \( \eta = \text{bdd}(k, l, u) \) 定义其“定义约束”：
    \[
    v_\eta \Rightarrow \sum_{i\ge k} v_i b_i \le l, \quad
    v_\eta \Leftarrow \sum_{i\ge k} v_i b_i \le u.
    \]
  - 利用子节点的定义约束，通过**冗余性规则**引入两个重言变量，再通过分情况推导（对 \( b_k \) 真/假）证明两者等价，从而导出父节点的定义约束（命题 10）。
  - 对于因 BDD 的归约性而造成的节点复用（即同一节点对应不同 \( k \)），可通过线性步骤从已有节点推导出新约束（命题 11）。
  - 对于 MDD，分情况数扩展为 \( |I|+1 \)，利用已发现的互斥约束确保恰有一种情况成立。

## 3. 实验设计

- **数据集**：使用 **MaxSAT Evaluation 2024** 的所有实例，包含未加权（unweighted）和加权（weighted）两类。
- **基准求解器**：**WMaxCDCL**（Coll et al., 2025b），当前最先进的分支定界 MaxSAT 求解器（在 2024 未加权赛道和 2023 加权赛道作为组合求解器获胜）。
- **对比方法**：
  - 无证明日志的原始求解 vs. 开启证明日志的求解（即本文实现的带证明输出的版本）。
  - 证明检查时间：使用 **VeriPB** 检查器（时间限制 10 小时，内存 64 GB）。
- **硬件环境**：2×32 核 AMD EPYC 9384X 处理器（每个求解调用分配单核），timeout = 1 小时，memory = 32 GB。

## 4. 资源与算力

- **计算资源**：所有实验在 CPU 单核上运行（每核独立），未使用 GPU。求解时无并行。
- **训练**：不涉及训练，只做推理求解。
- **文中明确说明时间/内存限制**：求解 step 时间 1 小时，内存 32 GB；检查 step 时间 10 小时，内存 64 GB。

## 5. 实验数量与充分性

- **求解实验**：共 **701 个**实例被无证明日志求解器成功求解；其中 **695 个**实例在启用证明日志后也成功求解（6 个无法求解）。
- **检查实验**：对 695 个生成的证明尝试检查，**485 个**成功通过，**202 个**超时，**8 个**内存溢出。
- **对比维度**：求解时间（有/无证明日志）散点图（Figure 1），证明检查时间 vs 求解时间散点图（Figure 2）。
- **充分性分析**：实验覆盖了 2024 年 MaxSAT 评测全集，具有很强的代表性。但未进行消融实验（如单独测量前瞻、MDD 编码的贡献），也未与其他证明日志方案（如文献中已有的基于解改进或核心引导的方案）直接对比（因范式不同）。作者诚实指出了检查时间过高的局限性，未来可优化检查器和证明格式。

## 6. 主要结论与发现

1. **可行性**：首次成功为分支定界 MaxSAT 求解（含前瞻和 MDD 编码）实现了完整的证明日志。
2. **运行时开销**：中位数为 **19%**，大部分实例开销可控；但 **10% 的实例开销超过 4.61 倍**，原因可能是大目标变量数下定义约束的线性输出。
3. **证明检查性能**：中位数检查时间为求解时间的 **42.94 倍**，过高的检查成本是目前的主要瓶颈，有待后续优化（如更快的检查器 PBOxide、证明修剪工具、使用带提示的 RUP 等）。

## 7. 优点

- **理论创新**：将分支定界 MaxSAT 中的前瞻下界估计（加权局部核）与 VeriPB 证明系统衔接，给出了从核集到学习子句的切割平面推导（定理 5），并形式化了 MDD 编码等价性的证明方法（命题 10、11）。
- **实现完整性**：在真正的工业级求解器（MaxCDCL）上实现了多种高级技巧的证明日志，包括子句活化、等价文字检测、子句包含简化等。附录中声称包含所有相关技术。
- **实验客观性**：使用了官方评测基准，时间限制和内存限制明确，散点图直接展示了每一点结果，无选择性报告；同时指出了检查时间的不足，未回避缺陷。

## 8. 不足与局限

- **检查效率低**：超过 1/3 的证明未被成功检查（210/695），其中大部分超时。检查时间比求解时间高两个数量级，实际应用价值受限。
- **特定场景性能退化**：对于目标变量多的实例，MDD 编码的证明生成可能引入巨大开销（定义约束线性输出）。未提供缓解措施。
- **未涵盖全部前沿技术**：作为未来工作，未包含最近提出的“文字解锁（literal unlocking）”技术；部分预处理器（如预处理 CNF）未详细说明认证细节。
- **实验缺乏消融分析**：未分别测量前瞻、MDD 编码、其他简化技巧各自的开销贡献，难以定位最大性能瓶颈。
- **比较公平性**：无直接对比其他 MaxSAT 证明日志方案（因范式不同），但本文属于填补空白，这可以接受。
- **可重复性**：作者已提供 GitHub 仓库（论文中引用 Zenodo 链接），但未提及是否提供脚本或 Docker 环境，理论上可重复。

（完）
