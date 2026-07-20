---
title: Massively Parallel Continuous Local Search for Hybrid SAT Solving on GPUs
title_zh: 面向GPU上混合SAT求解的大规模并行连续局部搜索
authors: "Yunuo Cen, Zhiwei Zhang, Xuanyao Fong"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/33211/35366"
tags: ["query:sat-bva-cnf"]
score: 8.0
evidence: 提出了在GPU上使用连续局部搜索的大规模并行混合SAT求解器，是一种新颖的SAT求解技术
tldr: 本文针对传统CDCL SAT求解器并行性受限的问题，提出了FastFourierSAT求解器。该求解器基于梯度驱动的连续局部搜索，并利用FFT算法高效计算对称多项式，从而在GPU上实现大规模并行。实验表明，该求解器在大量实例上取得了数量级的加速，为SAT求解开辟了新的并行化途径。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 传统CDCL求解器本质顺序，难以利用GPU并行计算能力。
method: 采用连续局部搜索框架，设计基于FFT的并行算法计算基本对称多项式。
result: 在GPU上实现了高度并行，求解速度相比CDCL求解器有显著提升。
conclusion: 混合连续局部搜索是高效并行SAT求解的有前景方向。
---

## Abstract
Although state-of-the-art (SOTA) SAT solvers based on conflict-driven clause learning (CDCL) have achieved remarkable engineering success, their sequential nature limits the parallelism that may be extracted for acceleration on platforms such as the graphics processing unit (GPU). In this work, we propose FastFourierSAT, a highly parallel hybrid SAT solver based on gradient-driven continuous local search (CLS). This is achieved by a parallel algorithm inspired by the fast Fourier transform (FFT)-based convolution for computing the elementary symmetric polynomials (ESPs), which is the major computational task in previous CLS methods. The complexity of our algorithm matches the best previous result. Furthermore, the substantial parallelism inherent in our algorithm can leverage the GPU for acceleration, demonstrating significant improvement over the previous CLS approaches. FastFourierSAT is compared with a wide set of SOTA parallel SAT solvers on extensive benchmarks including combinatorial and industrial problems. Results show that FastFourierSAT computes the gradient 100+ times faster than previous prototypes on CPU. Moreover, FastFourierSAT solves most instances and demonstrates promising performance on larger-size instances.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：传统基于冲突驱动子句学习（CDCL）的SAT求解器虽然工程上很成功，但其本质是顺序执行的——每步决策和传播依赖于历史状态，因此难以在GPU等并行计算平台上获得原生加速。现有并行SAT求解器多采用分治与组合（portfolio）方法，仅实现线程级并行，而指令级和数据级并行有限。
- **研究动机**：受机器学习中神经网络训练成功和GPU硬件进步的启发，希望利用连续局部搜索（CLS）框架将SAT问题转化为连续优化问题，并通过梯度下降求解。已有CLS方法（如FourierSAT、GradSAT）的梯度计算是主要性能瓶颈，且其理想并行执行时间仍为线性（O*(k)），难以充分利用GPU。
- **整体含义**：本文提出一种基于快速傅里叶变换（FFT）的高度并行算法来加速CLS中的核心计算——基本对称多项式（ESP）的求值，从而在GPU上实现大规模并行混合SAT求解器FastFourierSAT，为SAT求解提供了新的并行化途径。

### 2. 论文提出的方法论：核心思想、关键技术细节、算法流程

- **核心思想**：将SAT公式转换为多项式优化问题，目标函数为各约束Walsh展开（WE）的加权和。利用FFT将ESP的计算转化为向量化操作，使得梯度可以通过自动微分（Autodiff）高效并行计算。
- **关键技术细节**：
  - **FFT-based Evaluation**：对于长度为k的对称约束，其ESP可由k个序列[xi,1]的卷积得到。根据卷积定理，可通过三步计算：
    1. **加法（ADD）**：将每个序列做DFT，得到频域表示γ_i = [1+xi, ω+xi, …, ω^k+xi]^T（通过外加法实现）。
    2. **乘法（MUL）**：在频域按元素相乘，得到γ_c = ∏ γ_i。
    3. **乘法（MUL）**：乘以预计算的共轭Walsh系数f̃_c = f̂_c·W^{-1}，得到WE值。
  - **链式法则求导**：反向遍历计算图，复用前向中间结果，实现梯度计算，复杂度O(k²)，与前人最佳一致。
  - **三层次并行映射**：数据级并行（SIMD）→ warp；指令级并行（不同约束的WE计算独立）→ 同一SM中的warp；线程级并行（多初始点并行搜索）→ 多个SM。
- **算法流程**：
  - **Algorithm 1**：CLS框架，循环进行梯度下降局部搜索，若找到满足所有约束的点则返回，否则调整权重并重启。
  - **Algorithm 2**：FFT前向求值，对每个约束并行执行ADD、MUL、MUL三步，累加到总目标。
  - **Algorithm 3**：集成CDCL的Hybrid MaxSAT求解，先用CDCL求解硬约束得到部分模型，用来初始化随机点，再运行CLS优化，反复迭代。

### 3. 实验设计：数据集/场景、benchmark、对比方法

- **Benchmark 1**：梯度计算时间。生成m∈{100,200,400}个基数约束，长度l∈{8,16,32}，在10000个采样点上平均耗时。
- **Benchmark 2**：混合SAT公式。
  - 随机基数约束：N∈[50,250]条，每约束长度l=N/5，共100实例。
  - 带错误率的学习问题：N∈[20,60]，错误率e=1/4，m=2N个XOR约束，共100实例。
- **Benchmark 3**：图问题的混合MaxSAT（软XOR约束）。包括哈密顿回路（|V|∈{10,20,30,40,50}）和顶点着色（|V|∈{100,200,300,400,500}），各10个实例。
- **Benchmark 4**：SAT竞赛2023可满足实例（1000s超时）和MaxSAT评估2023未加权任意时间实例。
- **对比方法**：
  - CLS类：FourierSAT、GradSAT（CPU版）。
  - 并行DLS：PalSAT、WalkSAT。
  - 并行CDCL（CPU）：PRS、CryptoMiniSAT（CMS）。
  - 并行CDCL（GPU）：GpuShareSAT、ParaFrost（均有CPU+GPU版本）。
  - MaxSAT：LinPB、NoSAT-MaxSAT、Loandra、NuWLS-c等。
  - 所有对比均使用原始论文或官方实现，参数按推荐设置。

### 4. 资源与算力

- **CPU实验**：双AMD Epyc 7773X CPU（2TB RAM），每个实验使用32线程、128GB RAM。
- **GPU实验**：Intel i9-12900F CPU，32GB RAM，NVIDIA RTX 3080 Ti GPU。
- **超时**：默认300秒，Benchmark 4中部分实验为1000秒。
- **说明**：论文未明确提及训练时长或总GPU耗时，但给出了梯度计算时间的详细对比（表1）。

### 5. 实验数量与充分性

- **数量**：共4个Benchmark，覆盖合成随机实例和工业竞赛实例。Benchmark 1为微基准测试，Benchmark 2、3为可重复生成的多规模实例（各100/10组），Benchmark 4为真实公开竞赛数据。
- **充分性与公平性**：
  - 对比方法覆盖主流CPU和GPU并行求解器，且包含同类型的CLS基线。
  - 使用了PAR-2得分和任意时间得分等标准评价指标。
  - 对工业实例（Benchmark 4）单独分析，并指出了CLS在CNF编码下的性能下降（消融分析）。
  - 存在一定偏差：Benchmark 2、3的实例规模较小或合成性质强，对比时未进行统计显著性检验；消融实验仅部分提及（如不同初始化方法）。

### 6. 论文的主要结论与发现

- **梯度加速显著**：FastFourierSAT GPU版比GradSAT快472倍、比FourierSAT快1280倍（最大实例），CPU版也实现数十倍加速。
- **混合SAT求解竞争力强**：在所有基数约束实例上求解成功，parity learning问题中接近CPU版CryptoMiniSAT；图问题MaxSAT的任意时间得分优于多数对比方法（如Loandra、NuWLS-c）。
- **工业实例表现受限**：在SAT竞赛和MaxSAT评估实例上，FastFourierSAT虽能解部分实例（如SC'23中13个），但远不如竞赛级CDCL求解器（VBS得分0.954 vs 0.327）。
- **并行方案有效**：理想执行时间可达O*(log k)，充分利用GPU的三层次并行架构。

### 7. 优点：方法或实验设计上的亮点

- **理论创新**：提出基于FFT的并行ESP求值算法，复杂度O(k²)匹配最佳，但理想并行时间仅为对数级，为CLS提供了理论上的并行下限。
- **工程实现**：利用JAX等自动微分框架，一次性实现高效前向求值和反向梯度，代码简洁且易用GPU加速。
- **广泛对比**：不仅与同类CLS比较，还与多种最先进并行求解器（包括GPU版CDCL）对比，覆盖不同范式。
- **支持混合约束**：天然处理非CNF约束（基数、XOR等），避免编码开销，扩展了应用场景。

### 8. 不足与局限：包括实验覆盖、偏差风险、应用限制

- **数值稳定性**：频域计算存在舍入误差，虽通过高精度缓解但带来额外计算和内存开销，实际精度问题未深入分析。
- **不完备性**：CLS只能搜索解而不能证明不可满足，因此无法用于验证UNSAT实例；论文未讨论与完备方法的结合方式。
- **工业规模性能差距**：在真实竞赛WCNF实例上远落后于成熟CDCL求解器，且当公式经过CNF编码后性能急剧下降（如图问题变量数暴增）。
- **实验局限性**：
  - Benchmark 2、3的实例规模较小，缺少对更大规模（如数百万变量）工业问题的评估。
  - 消融实验不充分：仅对比了有无CDCL初始化，未系统分析并行搜索线程数、权重自适应、优化器类型等影响。
  - 公平性风险：对比的GPU CDCL求解器（GpuShareSAT、ParaFrost）在作者实验条件下对某些问题表现较差，可能受限于内存带宽或超时设置；未进行参数调优或运行多次取中位数。
- **硬件依赖**：方法强烈依赖GPU，且当前实现仅测试单块RTX 3080 Ti，多GPU或多卡扩展性未讨论。

（完）
