---
title: LLM-Guided Quantified SMT Solving over Uninterpreted Functions
title_zh: 大语言模型引导的带未解释函数的量化SMT求解
authors: "Kunhang Lv, Yuhang Dong, Rui Han, Fuqi Jia, Feifei Ma, Jian Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38445/42407"
tags: ["query:sat-bva-cnf"]
score: 4.0
evidence: SMT求解技术
tldr: 本文提出AquaForte框架，利用大型语言模型为带未解释函数的量化SMT公式的实例化提供语义指导，通过约束分离和结构化提示生成满足约束的候选函数定义，显著缩小求解器的搜索空间，提高了求解效率。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 传统量化实例化方法缺乏对未解释函数约束的语义理解，搜索空间过大。
method: 使用LLM生成语义指导的候选实例，并采用约束分离和结构化提示。
result: 有效减少搜索空间，提升求解器性能。
conclusion: LLM可以为SMT求解提供有效的语义引导。
---

## Abstract
Quantified formulas with Uninterpreted Functions (UFs) over non-linear real arithmetic pose fundamental challenges for Satisfiability Modulo Theories (SMT) solving. Traditional quantifier instantiation methods struggle because they lack semantic understanding of UF constraints, forcing them to search through unbounded solution spaces with limited guidance. We present AquaForte, a framework that leverages Large Language Models to provide semantic guidance for UF instantiation by generating instantiated candidates for function definitions that satisfy the constraints, thereby significantly reducing the search space and complexity for solvers. Our approach preprocesses formulas through constraint separation, uses structured prompts to extract mathematical reasoning from LLMs, and integrates the results with traditional SMT algorithms through adaptive instantiation. AquaForte maintains soundness through systematic validation: LLM-guided instantiations yielding SAT solve the original problem, while UNSAT results generate exclusion clauses for iterative refinement. Completeness is preserved by fallback to traditional solvers augmented with learned constraints. Experimental evaluation on SMT-COMP benchmarks demonstrates that AquaForte solves numerous instances where state-of-the-art solvers like Z3 and CVC5 timeout, with particular effectiveness on satisfiable formulas. Our work shows that LLMs can provide valuable mathematical intuition for symbolic reasoning, establishing a new paradigm for SMT constraint solving.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：带未解释函数（Uninterpreted Functions, UFs）的量化公式（QUFNIRA）在SMT求解中面临巨大挑战。传统方法（如E-matching、MBQI）仅依赖语法模式匹配，缺乏对UF约束的语义理解，导致搜索空间无界、实例化效率低下。
- **研究动机**：实际应用中UF常代表有明确数学含义的对象（如距离函数、成本函数），但现有求解器将其视为完全透明的符号，浪费了可利用的数学直觉。论文旨在利用大型语言模型（LLM）的数学推理能力，为UF实例化提供语义指导，从而缩小搜索空间，提升求解效率。
- **整体含义**：提出一种新颖的神经符号融合框架，将LLM的直觉推理与传统SMT求解的形式保证相结合，建立了一种新的SMT求解范式。

### 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程

- **核心思想**：通过LLM分析UF在约束中的使用模式，生成具体的数学定义（如多项式、分段函数）来替换抽象UF，进而将量化问题转化为可直接求解的实例化公式。
- **关键技术细节**：
  - **预处理**：公式重写（简化、展开解释函数） + 约束分离（利用并查集将共享UF的约束划分为独立组件），降低单次LLM查询复杂度。
  - **LLM实例化**：为每个组件构造结构化提示（包含任务指令、数据、输出格式、提示信息），要求LLM以JSON格式输出推理过程、置信度和具体函数定义（符合SMT-LIB语法）。
  - **后处理**：验证LLM输出（语法检查、类型检查），无效则重新查询；合并所有组件的实例化结果，维护全局符号表避免冲突。
- **核心算法流程**：
  - **Algorithm 1（公式重写与纯化）**：输入公式φ和UF集合F，输出独立组件C。步骤：重写公式 → 用并查集合并共同出现在约束中的UF → 按代表元分组约束。
  - **Algorithm 2（自适应LLM引导的SMT求解）**：循环最多N次或超时T。每次：带历史失败信息查询LLM获得实例化I和触发模式T → 应用实例化并添加触发器 → 调用有界SMT求解器（超时τ1） → 若SAT则返回；若UNSAT则生成排除子句¬(fi=Ii)并加入历史和学习子句；超时则记录失败。最终回退：调用完整SMT求解器处理原公式加上所有学习子句。

### 3. 实验设计：使用的数据集、Benchmark、对比方法

- **数据集**：
  - **SMT-COMP 2025基准**：完整UFNIRA和UFLRA实例，共281个。
  - **自定义SOS验证数据集**：600个多项式平方和分解问题（判断能否表示为恰三个平方和），维度n∈{1,2,3}，源多项式数m∈{1,2,3,4}（m≤3可满足，m=4压缩挑战）。
  - **自定义数学函数数据集（MFD）**：600个随机生成的UFNIRA实例，涵盖有理函数不等式、分段函数不等式、递归函数、函数极限等四类，每类150个。
- **总规模**：1481个实例。
- **对比方法**：
  - 基线求解器：Z3 (v4.15.0)、CVC5 (v1.2.1)。
  - LLM模型：GPT-4.1、DeepSeek-V3、Claude-4-Sonnet（温度0.01）。
  - CVC5自身策略：MBQI、CEGQI、枚举（ENUM）等合成方法。
  - 超时设置：24秒（主）和1200秒。
  - 迭代次数：默认N=1，额外实验N=1~10。

### 4. 资源与算力

- **计算环境**：服务器配备AMD EPYC 7763 64核处理器（2.45 GHz）和512 GB RAM。未明确说明使用了GPU（LLM推理可能通过API，不需要本地GPU；论文未提及GPU型号、数量或训练时长）。
- **LLM推理时间**：平均每次查询GPT-4.1耗时7.6秒，Claude-4-Sonnet 16.56秒，DeepSeek-V3 20.66秒。
- **总体算力评估**：论文未披露LLM训练算力或SMT求解器运行的具体资源消耗，仅强调实验在统一硬件上进行，保证一致性。

### 5. 实验数量与充分性

- **数量**：共3个benchmark套件，1481个实例；对比了2个SOTA求解器、3个LLM、2种超时策略、10种迭代次数；对cvc5还做了4种内部策略的对比。
- **充分性**：实验设计较为系统，涵盖了标准基准和定制数学问题，测试了不同场景（SAT/UNSAT、不同LLM、不同迭代、不同超时）。消融实验（迭代次数影响、不同LLM互补）增强了结论的可靠性。
- **公平性**：所有实验在同一硬件上运行，超时设置遵循SMT-COMP惯例（24秒）；LLM使用固定温度0.01确保确定性；统计了“虚拟最佳”组合，有效消除了单一LLM偏差。但未提及多次运行取平均（可能是单次确定性运行），结果的可重复性较好。

### 6. 论文的主要结论与发现

- **总体效果**：LLM引导大幅提升求解性能。与Z3基线相比，AF+Z3（GPT-4.1）解决实例数量提升80.0%（436→785）；与CVC5基线相比提升183.6%（226→641）。Claude-4-Sonnet最佳：Z3提升90.6%（436→831）。虚拟最佳（所有LLM联合）达到897（Z3）和763（CVC5），提升105.7%和237.6%。
- **SAT/UNSAT不对称性**：SAT实例提升约3.6倍，UNSAT实例提升极小。表明LLM擅长通过语义识别给出满足性解释，但难以构造穷举不满足性证明。
- **多迭代效果**：从1次迭代增加到10次，平均提升Z3 15.5%、CVC5 17.6%，但5~7次后收益边际递减。
- **长时间超时效果有限**：超时从24秒增至1200秒，基线和LLM方法提升均不超过1%，说明问题难度不主要由时间决定。
- **不同LLM互补**：不同LLM在不同实例上各有擅长，联合使用（虚拟最佳）优于单个最佳。

### 7. 优点：方法或实验设计上的亮点

- **创新性融合**：首次系统性地将LLM的语义理解引入SMT量化实例化，突破了纯语法方法的局限，同时通过排除子句和回退机制保证了求解的正确性和完备性。
- **预处理设计**：约束分离显著降低LLM单次查询复杂度（从O(|F|²)到O(|Ci|²)），且保留了数学结构，提高了LLM输出质量。
- **结构化提示与验证**：强制LLM输出JSON格式并包含推理链，便于可解释性和错误回溯；后处理中的语法/类型校验确保了系统鲁棒性。
- **自适应迭代与学习**：失败历史被编码为排除子句，逐步缩小搜索空间，且回退到原求解器不削弱基础能力。
- **实验全面且消融到位**：涵盖标准基准与自定义难题，对比多种LLM和求解器，分析迭代次数和超时影响，并提供了虚拟最佳上界，论证了互补性。

### 8. 不足与局限

- **SAT偏向性**：对UNSAT问题几乎没有提升，限制了方法在验证反例发现等需要不满足性证明的场景中的应用。
- **LLM推理开销**：平均每次查询耗时7~20秒，在多迭代场景下可能成为瓶颈；论文未讨论如何加速LLM推理或使用更轻量级模型。
- **依赖LLM质量**：方法表现高度依赖于所选LLM的数学推理能力，对不擅长数学的LLM可能效果不佳；未探索开源模型的适用性（仅测试了闭源模型）。
- **扩展性未知**：当前仅针对UF与非线性算术的组合，未探讨对更复杂理论（如数组、位向量）或更大量词交替的适应性。
- **实验细节缺失**：未提供多次运行的标准差或置信区间，也未说明SMT求解器内部参数是否统一；LLM调用成本（API费用）未量化。

（完）
