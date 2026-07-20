---
title: "DiverSAT: A Novel and Effective Local Search Algorithm for Diverse SAT Problem"
title_zh: "DiverSAT:一种新颖有效的多样性SAT问题局部搜索算法"
authors: "Jiaxin Liang, Junping Zhou, Minghao Yin"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/33228/35383"
tags: ["query:sat-bva-cnf"]
score: 8.0
evidence: SAT求解技术、局部搜索
tldr: 本文提出DiverSAT局部搜索算法求解多样性SAT问题，引入三种启发式策略和扰动机制，在硬件验证、物流规划等公开基准上优于现有算法，为SAT求解技术提供了新的多样性导向方法。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 实际应用中用户常需要一组多样性的可满足赋值，而不仅仅是单个解。
method: 基于局部搜索框架，设计三种启发式和一种扰动策略以增强多样性。
result: 在多个领域的公开基准上优于现有算法。
conclusion: DiverSAT是解决多样性SAT问题的有效方法。
---

## Abstract
For many real-world problems, users are often interested not only in finding a single solution but in obtaining a sufficiently diverse collection of solutions. In this work, we consider the Diverse SAT problem, aiming to find a set of diverse satisfying assignments for a given propositional formula. We propose a novel and effective local search algorithm, DiverSAT, to solve the problem. To cope with diversity, we introduce three heuristics and a perturbation strategy based on some relevant information. We conduct extensive experiments on a large number of public benchmarks, collected from semiformal hardware verification, logistics planning,  and other domains. The results show that DiverSAT outperforms the existing algorithms on most of these benchmarks.

---

## 论文详细总结（自动生成）

## 1. 核心问题与整体含义（研究动机和背景）
- **研究问题**：Diverse SAT（多样性SAT）问题，即给定一个命题逻辑公式和一个正整数k，寻找一组包含k个可满足赋值（模型）的解集，使得所有成对汉明距离之和最大化。
- **背景与动机**：许多实际应用（如智能规划、软件测试、半形式化硬件验证等）对解的多样性有要求，而传统SAT求解器重复运行倾向于返回相似解。现有专门算法（如Nadel提出的Diverse kSet）基于CDCL调整变量排序，但解的质量仍有提升空间。因此需要更高效的算法来生成多样化解集。

## 2. 方法论：核心思想、关键技术细节、算法流程
- **整体框架**：DiverSAT是一种局部搜索算法，包含**初始化阶段**和**优化阶段**。先通过子算法GSMD生成k个初始模型构成解S，然后反复尝试替换S中的一个模型以提升多样性。
- **GSMD（Generate a Single Model with Diversity）子算法**：
  - **初始候选模型构建**：以概率pr随机生成初始赋值；否则采用引导机制，利用变量决策启发式和极性选择启发式逐个赋值，并执行单元传播；若发生冲突，根据已赋值变量比例决定忽略冲突或回溯。
  - **变量决策启发式**：对每个变量x定义得分 score(x) = λ1 × PRE_Order(x) + λ2 × VSIDS(x)，其中PRE_Order是x在最近找到模型中的赋值顺序（靠后得分高），VSIDS是活动度。采用BMS策略（随机选m个变量，选得分最高的）。
  - **极性选择启发式**：为每个变量x维护极化偏好pol(x)，表示赋值为1的概率。初始化与更新规则视阶段而定：初始化阶段pol(x)初始0.5，根据赋值增减1/(2k)；优化阶段pol(x)初始化为1减去在S中取1的比例，然后根据替换模型中的赋值变化调整±1/k。
  - **局部搜索阶段**：迭代翻转变量直到找到模型或达到步数上限。使用**SCCA（强化配置检查与期望）启发式**选择翻转变量。SCCA是CCA的变体，考虑增益gain(x)=make(x)-break(x)（基于动态权重），并优先选择增益为正且配置已变的变量，或增益大于平均权重且配置未变的变量，平局时优先选翻转偏好fp(x)大的变量。fp(x)依据pol(x)和当前赋值计算：若α(x)=0则fp(x)=pol(x)，否则fp(x)=1-pol(x)。
- **优化阶段（UpdateSolution）**：
  - 定义模型的**得分**为它与其他k-1个模型平均汉明距离，反映独特性。
  - 每次迭代调用GSMD获得一个新模型α，加入S得到S'。若迭代计数器cnt<max_cnt，则找出S中得分最低的模型α_worst，若score(α,S')>score(α_worst,S')则替换掉α_worst；否则进入自适应扰动：当DQ提升小于0.5%持续超过max_cnt次时，将α替换掉离它最近的模型α_close（而非α_worst），以帮助跳出局部最优。
- **关键创新**：三种启发式（变量决策、极性选择、SCCA）和自适应扰动机制协同作用，增强多样性。

## 3. 实验设计：数据集、基准、对比方法
- **基准**：
  - 半形式化硬件验证基准（66个实例，来自Nadel主页）。
  - SATLIB应用基准（2670个实例），筛选模型数>100的实例，涵盖17种问题类型（物流规划、有界模型检验、电路故障分析等）。具体包括ais、bmc、flat(不同规模)、GCP、hardware、II、logistics、qg、ssa、SW100-8等。
- **对比方法**：
  - **Diverse kSet**（Nadel 2011，PB-CPGUIDE 100-VRANDGLOB 30启发式），基于Maple LCM实现（因未获得源码）。
  - **randAllSAT**：基于回溯AllSAT求解器+随机分支启发式，生成k个模型。
- **参数设置**：k=10、50、100；每个实例运行10次，截止时间3600秒。参数pr=0.15、λ1=100、λ2=1、m=50、r=0.7、max_step=2×10^5、max_cnt=20。

## 4. 资源与算力
- 文中仅提及实验环境：Intel(R) Xeon(R) 2.20 GHz CPU、10 GB内存、CentOS 6.10。**未使用GPU**，也未提及训练时长（因为无需训练）。算法为局部搜索，每次运行有超时限制。

## 5. 实验数量与充分性
- 总实例数：2736个（66+2670），但实际根据模型数筛选，最终使用全部。每个实例针对三个k值各运行10次，共计约82180次运行。
- 对比实验：DiverSAT vs Diverse kSet vs randAllSAT，报告平均DQ和平均运行时间。
- 消融实验：比较DiverSAT、DiverSAT\H（去除三种启发式，改用VSIDS+CCA）和DiverSAT\M（去除自适应扰动机制）。在相同基准集上进行，报告平均DQ和运行时间（图与表）。
- 充分性评价：实验覆盖多种领域、不同规模（变量数从90到21万+，子句数从300到73万+）；对比方法为当前最好；消融实验验证了各组件的必要性。但未报告方差或统计检验，且只给了平均结果，未提供详细分布。

## 6. 主要结论与发现
- DiverSAT在所有基准上均获得比Diverse kSet和randAllSAT更高的平均多样性质量（DQ），尽管运行时间更长。
- 随着k增大，DiverSAT的优势更明显，尤其在大规模实例（如bmc、hardware）上提升显著。
- 消融实验表明：三个启发式和自适应扰动机制均对性能有贡献，且彼此互补；DiverSAT在运行时间上并未显著劣于简化版本，说明开销可接受。
- 验证了局部搜索方法在多样性SAT问题上的有效性。

## 7. 优点
- **方法新颖**：首次系统地将局部搜索应用于多样性SAT问题，提出了基于统计信息的变量决策和极性选择启发式，有效引导搜索空间。
- **技术设计合理**：SCCA在经典CCA基础上融入翻转偏好，兼顾循环避免与多样性保持；自适应扰动机制避免过早陷入局部最优。
- **实验充分**：使用大量公开基准（包括硬件验证、规划等），对比了当前主流算法，并进行了彻底的消融分析。
- **结果显著**：在所有问题类型上平均DQ优于基线，且在大规模问题中优势突出，说明算法具有良好的可扩展性。
- **开源支持**：数据集和代码（推测）在GitHub提供，便于复现。

## 8. 不足与局限
- **运行时间较长**：DiverSAT在多数实例上耗时多于对比算法，尤其当k较大时（如k=100，硬件基准平均1025秒），可能不适用于对实时性要求高的场景。
- **参数依赖**：多个参数（pr、λ1、λ2、m、r、max_step、max_cnt）需手动调节，文中仅基于初步实验确定，未进行系统敏感性分析，最优性存疑。
- **实现偏差风险**：Diverse kSet和randAllSAT为作者自行实现（基于Maple LCM和AllSAT求解器），可能与原始方法存在差异，对比公平性需谨慎解读。
- **统计显著性缺失**：仅报告平均值，未提供标准差、箱线图或统计检验，无法判断差异是否显著。
- **理论分析不足**：未给出算法的近似比或收敛性证明，性能依赖经验。
- **未对比最新统一采样方法**：如tabularAllSAT等，虽然文中解释了因输出格式不同未对比，但仍是潜在局限性。
- **应用限制**：仅评估了CNF公式，未考虑其他范式或约束满足问题；且多样性定义局限于汉明距离，实际中可能有更复杂的多样性度量。

（完）
