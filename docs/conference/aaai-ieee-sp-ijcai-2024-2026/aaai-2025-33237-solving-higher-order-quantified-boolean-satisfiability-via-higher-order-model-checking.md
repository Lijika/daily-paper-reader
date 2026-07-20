---
title: Solving Higher-Order Quantified Boolean Satisfiability via Higher-Order Model Checking
title_zh: 通过高阶模型检查求解高阶量化布尔可满足性
authors: "Hiroshi Unno, Takeshi Tsukada, Jie-Hong Roland Jiang"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/33237/35392"
tags: ["query:sat-bva-cnf"]
score: 4.0
evidence: 通过模型检查求解高阶QBF
tldr: 本文提出了首个高阶量化布尔公式（HOQBF）求解器，通过归约到高阶模型检查来求解k-EXPTIME问题。这是SAT求解技术的重大进步，但未直接关注BVA或CNF预处理。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 缺乏高阶QBF求解器，限制了对k-EXPTIME问题的处理。
method: 将HOQBF归约到高阶模型检查问题。
result: 实现了首个HOQBF求解器，能处理高复杂度问题。
conclusion: 拓展了布尔可满足性的范畴，但与BVA等技术方向不同。
---

## Abstract
The satisfiability (SAT) problem of higher-order quantified Boolean formula (HOQBF) emerged as a natural generalization of SAT, quantified SAT, and second-order quantified SAT. 
 It allows succinct encoding of k-EXPTIME problems beyond the reach of prior Boolean satisfiability formulations, but its application was hampered by the lack of solvers.  In this paper, we present the first HOQBF solver that leverages techniques from the model-checking community.  Our HOQBF solver is based on reduction to higher-order model checking, which is a generalization from model checking of while-programs to that of higher-order functional programs.  The ability of a higher-order model checker to deal with higher-order functions in a program is used to reason about higher-order quantifiers in HOQBF.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **研究动机**：高阶量化布尔公式（HOQBF）是SAT、量化SAT（QBF）和第二阶量化SAT（SOQBF）的自然推广，能够简洁编码k-EXPTIME问题，此前没有可用的求解器，限制了其实际应用。
- **背景**：HOQBF的满足性问题（HOSAT）是TOWER完全的，可表达高阶布尔函数变量上的量词。潜在应用包括Presburger算术、内存一致性验证、多Agent规划、安全系统综合等。
- **挑战**：此前仅存在理论复杂性分析，缺乏实际可运行的求解工具。

## 2. 论文提出的方法论
- **核心思想**：将HOSAT问题归约到高阶模型检查（Higher-Order Model Checking）问题。利用高阶模型检查器处理程序中的高阶函数的能力，来推理HOQBF中的高阶量词。
- **关键技术细节**：
  - 对HOQBF公式φ构造一个高阶函数程序Pφ，该程序返回true当且仅当φ可满足。
  - 量词翻译策略：
    - 一阶布尔量词（∀x:bool, ∃x:bool）通过递归函数枚举所有布尔值实现。
    - 高阶量词（∀f:A, ∃f:A）通过定义“枚举结构”（enumeration structure）实现，为每个类型A提供零元素、最大值判断和后续函数，从而用循环逻辑模拟全部元素的枚举。
    - 枚举结构递归定义：bool类型直接定义；函数类型（bool→B或一般C→B）通过将函数视为基为N+1的多位数字来定义零、后继和最大值判断。
  - 翻译是线性的，将n阶公式映射到n+1阶模型检查问题。
- **Skolem函数提取**：当模型检查器回答“是”时，其输出的见证（基于归约交集类型系统的派生树）可用于提取Skolem函数，通过分析子程序Pφ的规范（specification）确定各存在量化变量的取值。

## 3. 实验设计
- **数据集/基准**：自建HOQBF基准集，包含21个问题实例。涵盖多种场景：
  - 基本高阶公式（如例1、例2）。
  - 二元关系性质（对称性、反对称性、自反闭包、对称闭包、传递闭包、左/右全、左/右唯一等）。
  - Schröder-Bernstein定理的HOQBF编码。
  - CPS变换后的函数存在性问题（arity1, arity2）。
- **输入格式**：支持DIMACS、QDIMACS、DQDIMACS以及自定义HOQBF格式。
- **对比方法**：无对比基线（因为是首个HOQBF求解器），仅呈现自身结果。

## 4. 资源与算力
- **硬件**：12th Gen Intel Core i7-1270P 2.20 GHz处理器，32 GB内存。
- **GPU**：未提及使用GPU。
- **训练时长**：不涉及训练，仅运行求解。单次求解超时设置为300秒。

## 5. 实验数量与充分性
- **实验数量**：21个问题实例，每个实例记录阶数、变量数、量词交替数、结果（SAT/UNSAT/超时/内存溢出）及耗时。
- **充分性评估**：
  - 覆盖了不同阶数（1阶、2阶、3阶、4阶）、不同量词交替深度、不同变量数的问题。
  - 包含可满足与不可满足实例。
  - 测试了求解器的极限（如sym-asym-bb超时，sym-cl-uniq、trans-cl-uniq、cps-arity2内存溢出）。
  - **不足**：基准集较小（21个），未与任何其他方法对比（因无其他求解器），也未进行消融实验（如不同后端模型检查器、不同枚举结构实现的影响）。

## 6. 论文的主要结论与发现
- **成功实现**：首次开发出HOQBF求解器HOMCSAT，能处理多种高阶量词和复杂逻辑问题。
- **性能**：在大多数小规模实例上快速（<1秒），部分中等规模实例（如refl-cl-uniq耗时124秒）仍可求解。
- **瓶颈揭示**：后端高阶模型检查器HOR SAT 2在处理包含大量分支的问题时，交集类型搜索空间膨胀，导致超时或内存耗尽。
- **改进方向**：建议引入BDD/ZDD压缩交集类型表示，以及借鉴SAT/QBF中的抽象、剪枝、传播技术。

## 7. 优点
- **方法创新**：首次将高阶模型检查应用于布尔可满足性问题，跨领域融合思路新颖。
- **翻译高效**：线性时间归约，且能保持阶数关系（n阶公式归约为n+1阶模型检查）。
- **功能完整**：不仅判断可满足性，还支持从模型检查见证中提取Skolem函数。
- **开源**：代码公开，便于复现和扩展。
- **问题覆盖全面**：基准实例涵盖不同阶数、量词交替、关系性质、经典定理等，展示了求解器能力。

## 8. 不足与局限
- **实验规模小**：仅21个实例，未在大规模或工业级问题上验证。
- **无对比基线**：无法客观评估相对于理论最优解或未来替代方法的相对性能。
- **后端性能瓶颈**：HOR SAT 2对分支多的HOQBF实例处理不佳，导致超时或内存溢出，限制了求解器可解问题的规模。
- **缺乏消融研究**：未分析枚举结构不同实现、不同模型检查后端、不同翻译优化对性能的影响。
- **应用限制**：虽然提出应用前景，但未在具体实际场景（如内存模型验证）中展示端到端结果。
- **算力信息不全**：仅提及CPU和内存，未说明是否有并行或分布式支持。

（完）
