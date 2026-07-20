---
title: "Verified Certificates via SAT and Computer Algebra Systems for the Ramsey R(3,8) and R(3,9) Problems"
title_zh: "通过SAT和计算机代数系统验证拉姆齐R(3,8)和R(3,9)问题的可验证证书"
authors: "(PDF |   Details)"
date: 2025-08-01
pdf: "https://www.ijcai.org/proceedings/2025/0292.pdf"
tags: ["query:sat-bva-cnf"]
score: 4.0
evidence: 使用SAT求解验证拉姆齐问题
tldr: "本文利用SAT求解器和计算机代数系统验证了拉姆齐数R(3,8)和R(3,9)问题，通过生成可验证的证书来证明结果。该方法展示了SAT求解技术在组合数学问题中的强大应用，但并未涉及BVA或CNF预处理等核心主题。"
source: IJCAI-2025-Accepted
selection_source: conference_retrieval
figures_json: "[{\"url\": \"assets/figures/ijcai-2025-accepted/ijcai-2025-292/fig-001.webp\", \"caption\": \"\", \"page\": 0, \"index\": 1, \"width\": 804, \"height\": 677, \"label\": \"Figure\"}]"
tables_json: "[{\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-292/table-001.webp\", \"caption\": \"\", \"page\": 0, \"index\": 1, \"width\": 618, \"height\": 345, \"label\": \"Table\"}, {\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-292/table-002.webp\", \"caption\": \"\", \"page\": 0, \"index\": 2, \"width\": 743, \"height\": 187, \"label\": \"Table\"}, {\"url\": \"assets/tables/ijcai-2025-accepted/ijcai-2025-292/table-003.webp\", \"caption\": \"\", \"page\": 0, \"index\": 3, \"width\": 1645, \"height\": 149, \"label\": \"Table\"}]"
motivation: 验证拉姆齐数问题的精确结果需要强大的计算工具，SAT求解器被用于生成和检查证书。
method: "结合SAT求解器和计算机代数系统，生成R(3,8)和R(3,9)问题的可验证证书。"
result: "成功验证了R(3,8)和R(3,9)的已知结果，并提供了可重用的证书。"
conclusion: SAT和计算机代数系统的结合为解决经典组合问题提供了有效途径。
---

## Abstract
No abstract is available.

---

## 论文详细总结（自动生成）

# 论文《Verified Certificates via SAT and Computer Algebra Systems for the Ramsey R(3,8) and R(3,9) Problems》中文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **研究背景**：Ramsey数 R(p,q) 是图论中的经典问题，定义为最小的正整数 n，使得任何 n 个顶点的完全图的红/蓝边着色必然包含一个蓝色的 p-团或一个红色的 q-团。其中 R(3,8)=28 和 R(3,9)=36 虽然已通过计算得到，但之前的计算依赖 uncertifiable 的图枚举工具（如 NAUTY），缺乏形式化验证，存在潜在错误风险。
- **核心问题**：如何为 R(3,8) 和 R(3,9) 的穷举搜索部分生成可独立验证的证书（certificates），保证正确性和完备性。
- **整体含义**：本文首次为这两个 Ramsey 数提供了可独立检查的认证结果，增强了计算机辅助证明的可信度，并展示了 SAT 求解器与计算机代数系统（CAS）结合的巨大潜力。同时，R(3,8) 和 R(3,9) 是仅有的两个尚未通过证书验证的非平凡 Ramsey 数。

## 2. 方法论：核心思想、关键技术细节
- **核心思想**：采用 SAT+CAS 混合范式，利用 SAT 求解器（CaDiCaL）进行高效的冲突驱动子句学习（CDCL），同时利用计算机代数系统（CAS）动态检测部分赋值的非正则性（non-canonicity），并生成阻塞子句（blocking clause）以剪枝对称分支，实现无同构的图生成（orderly generation）。
- **关键技术细节**：
  - **编码**：将 Ramsey 问题编码为 CNF 公式，变量表示边颜色（蓝色为真，红色为假），约束所有 p-团至少有一条红边，所有 q-团至少有一条蓝边。
  - **静态对称性破缺**：添加 lexicographic 排序约束（按行比邻接矩阵），引入 O(n³) 个辅助变量和子句。
  - **动态对称性破缺（顺序生成）**：采用 Read-Faradžev 的 orderly generation 方法。定义 adjacency matrix 的规范形（canonical form）为所有顶点置换下 lexicographically 最小的矩阵。当 SAT 求解器生成一个部分赋值对应一个完整的上三角子矩阵时，CAS 检查该子矩阵是否 canonical；若不是，则通过 IPASIR-UP 接口注入一个阻塞子句，该子句禁止当前非 canonical 矩阵及其所有后代。
  - **规模约束**：利用已知理论结果（如 Graver-Yackel 界）添加顶点度数和边数的基数约束（使用 totalizer 编码），缩小搜索空间。
  - **并行化**：实现 cube-and-conquer 策略，使用 AlphaMapleSAT（基于 MCTS 的 lookahead cubing）作为 cubing 求解器，MathCheck（CaDiCaL+CAS）作为 conquering 求解器。在 cubing 过程中，每次分裂变量后用 CaDiCaL+CAS 进行简化（10,000 次冲突），使得 CAS 生成的阻塞子句也能影响 cubing 过程。当子实例的证明文件超过 7 GiB 时，再返回进一步 cubing。
- **公式与算法流程**：流程可用文字描述如下：
  1. 将 Ramsey 实例编码为 CNF。
  2. 使用 AlphaMapleSAT 进行 cube 生成：每次选择一个变量分裂，并用 CaDiCaL+CAS 快速简化，直到消除的变量数达到阈值（R(8,3): 120; R(9,3): 100）。
  3. 每个 cube 被 MathCheck 求解，同时 CAS 动态注入阻塞子句；求解过程中产生 DRAT 证书。
  4. 所有 cube 返回 UNSAT 则原实例 UNSAT。
  5. 验证阶段：使用 DRAT-trim 证书检查器，并将 CAS 生成的子句标记为可信（prefix ‘t’），通过独立 Python 脚本验证每个 CAS 子句确实对应一个非 canonical 矩阵（应用 witness 置换后得到 lexicographically 更小的矩阵）。

## 3. 实验设计
- **数据集/场景**：两个 Ramsey 问题实例：R(3,8)/R(8,3) 和 R(3,9)/R(9,3)。注意 R(3,k) 和 R(k,3) 在 CNF 编码中不同（前者含较多正文字，后者含较多负文字），且实验发现 R(k,3) 实例求解更快。
- **Benchmark**：对比方法为纯 SAT 求解器（CaDiCaL alone），不加 CAS 的动态对称性破缺。
- **对比结果**（表 2）：
  - 对于 k=7: CaDiCaL+CAS 仅需 14.3s (R(3,7)) 和 8.2s (R(7,3))，而 CaDiCaL alone 需 564.3s 和 220.7s。
  - 对于 k=8: CaDiCaL+CAS 在 112.1h 和 18.5h 内完成；CaDiCaL alone 在 7 天后超时。
- **实验充分性**：实验覆盖了 R(3,8)、R(8,3)、R(3,9)、R(9,3) 四个实例，对比了 sequential 和 parallel 版本，并测试了 R(3,7)/R(7,3) 作为小规模验证。消融上：测试了是否开启 full canonical check（开启后速度提升 2 倍），以及是否使用 cube-and-conquer。未单独测试静态对称性破缺的贡献，但已集成在所有实例中。

## 4. 资源与算力
- **R(8,3)**：使用 Dual Xeon Gold 6226 @2.7GHz 集群，24 个 CPU 核心并行。总 wall clock 约 8 小时（求解 6.2 小时，验证 6.2 小时，总 CPU 时间 22,388 秒）。
- **R(9,3)**：使用 Dual AMD Epyc 7713 @2.0GHz 集群，128 个 CPU 核心并行。总 wall clock 约 26 小时（求解 697,575 秒 CPU 时间，验证 473,874 秒 CPU 时间）。
- **证书大小**：R(8,3) 证书共 5.8 GiB；R(9,3) 证书共 289 GiB。未提及 GPU。论文未显式说明总体算力成本（如总 CPU 小时数），但给出了各部分时间统计。

## 5. 实验数量与充分性
- **实验数量**：核心实验为 4 个实例的求解与验证。此外，对 R(3,7) 和 R(7,3) 进行了小规模对比测试。消融实验包括：对比纯 SAT、对比 sequential vs parallel、对比 partial vs full canonical check。未进行参数化超参数扫描（如 cubing 阈值的选择缺乏系统性消融）。
- **充分性与公平性**：
  - 对比方法（CaDiCaL alone）在 k=8 时超时，说明 SAT+CAS 显著更优。
  - 实验环境统一，但 k=8 和 k=9 使用不同 CPU 型号，可能影响直接对比。
  - 验证阶段独立完成，保证了结果的可信度。

## 6. 主要结论与发现
- **R(3,8)=28 和 R(3,9)=36** 的结论被成功验证，并生成了可独立检查的证书。
- SAT+CAS 方法相比纯 SAT 提速数个数量级：对于 k=8，sequential SAT+CAS 可在 18.5 小时内解决 R(8,3)，而纯 SAT 7 天超时。
- 并行 cube-and-conquer 进一步降低了 wall clock 时间，且总 CPU 时间也少于 sequential（R(8,3) 总 CPU 时间 22,388s vs sequential 66,504s），体现了更好的计算效率。
- 对于 R(9,3)，利用理论引理（Graver-Yackel）将问题简化为验证 (8,3;27;271) 图不存在，再通过并行 SAT+CAS 成功证明，最终确认 R(3,9)=36，减少了对原始证明中未验证的 (3,8;27;80) 图枚举的依赖。

## 7. 优点
- **可验证性**：首次为 R(3,8) 和 R(3,9) 提供可独立检查的证书，增强了计算的正确性。
- **高效率**：SAT+CAS 通过动态对称性破缺极大剪枝，性能远超纯 SAT。
- **通用性**：方法（orderly generation + IPASIR-UP）可扩展至其他组合问题；并行框架已模块化。
- **理论结合计算**：利用已知理论结果（度数和边数界）缩小搜索空间，降低了计算难度。
- **验证流程严谨**：不仅对 UNSAT 结果进行 DRAT 验证，还单独验证了 CAS 生成的每个阻塞子句的正确性，不依赖 CAS 的正确性。

## 8. 不足与局限
- **非完全形式化证明**：证书仅保证穷举搜索的正确性，但 SAT 编码本身未经形式化验证，因此论文明确指出这并非完整的形式化证明。缺少将 SAT 结果与 Ramsey 数定义在 Lean 或 Coq 中连接的工作。
- **实验覆盖有限**：仅验证了两个（对称关系下四个）已知结果，未尝试开放问题如 R(3,10)、R(4,6) 等。方法在更大规模上的可扩展性未知（证书已达 289 GiB）。
- **资源消耗大**：R(9,3) 需要 128 核和 289 GiB 证书，存储和验证成本高，对于更大实例可能不可行。
- **缺乏超参数系统调优**：cubing 停止阈值（消除变量数）、证书大小上限（7 GiB）等选择基于经验，未做充分消融研究。
- **CAS 开销占比**：R(9,3) 中 CAS 占求解时间的 38%，对于更复杂的图枚举可能成为瓶颈。
- **公平性细节**：对比纯 SAT 时未说明纯 SAT 是否也使用了静态对称性破缺（仅动态对称性破缺不同），可能影响对比的绝对公平性。

（完）
