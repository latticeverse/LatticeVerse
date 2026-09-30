<div align="center">

  <img src="assets/logo.png" alt="LatticeVerse" width="350">

  <p><strong>面向计算晶格建模、物理仿真、逆向设计和制造约束优化的统一研究框架与集成层。</strong></p>

[English](README.md) · [研究框架图](#如何阅读研究框架图) · [项目](#1-几何建模) · [数据集](#2-晶格数据集) · [引用](#引用)

</div>

## 这个仓库是什么

LatticeVerse 是一组晶格设计论文及其实现的项目总仓库。它记录各个项目之间的关系，定义共享的数据规范和集成约定，并链接到存放可执行研究代码的独立项目仓库。

整个仓库遵循一条端到端路径：

```text
几何建模
  -> 晶格数据集
  -> 物理仿真与评估
  -> 生成与优化
  -> 验证、制造与数据集扩展
```

各模块交换的基本单元是带版本信息的晶格样本。一个样本包含几何、材料、物理场、等效属性、设计目标、优化溯源信息和制造检查结果。这样，生成器可以提出结构，求解器可以评估结构，下游应用也可以复现这一决策过程。

除非项目被标记为已集成，否则本仓库不包含该项目的完整实现。权威的代码和实验细节请以对应的项目仓库、发布版本或论文为准。

## 如何阅读研究框架图

<p align="center">
  <img src="assets/pipeline.svg" width="100%" alt="LatticeVerse 研究框架图：几何建模、晶格数据集、物理仿真与评估、生成与优化，以及面向制造的应用">
</p>

这张研究框架图应从左向右阅读。几何建模生成参数化晶格单元，并将它们组织为晶格数据集。物理仿真计算局部物理场和等效属性，评估模块检查求解器精度和候选结构性能。这些结果为三个下游分支提供训练和优化输入：属性驱动的逆向设计、面向应用的优化，以及制造优化。通过筛选的候选结构可进一步验证、记录，并反馈到数据集以进一步扩展。

- 几何建模提供参数、标准化晶胞表示，以及网格或体素数据引用。
- 数据集构建将这些输出转换为带清单、数据划分和溯源信息的可复现样本。
- 仿真生成局部位移、应力、通量或温度场，以及等效属性。
- 评估比较数值求解器和学习型求解器，检查物理一致性，并对生成的候选结构进行排序。
- 生成与优化使用目标、约束和已评估样本生成新的候选结构。
- 通过验证的候选结构带着求解器结果、优化目标、约束和制造状态返回数据集。

## 1. 几何建模

这些项目定义进入晶格数据集的可控设计空间。每个适配模块都应导出标准化的几何表示，并保留能够重新生成该几何的参数。

| 年份 / 期刊或会议 | 项目 | 项目贡献 | 资源 |
|:--|:--|:--|:--|
| 2023 · [AM](https://www.sciencedirect.com/journal/additive-manufacturing) | **PPL** | 统一的参数化板晶格表示，支持直接四边形网格划分和基于水平集的形状优化，从而获得定制化的力学性能。 | [论文](https://doi.org/10.1016/j.addma.2023.103626) ·  [代码](https://github.com/latticeverse/ParametricPlateLattice) · [引用](docs/citations/ppl.bib) |
| 2022 · [AM](https://www.sciencedirect.com/journal/additive-manufacturing) | **PSL** | 基于骨架驱动的参数化壳晶格表示，能够控制拓扑和形态，并结合形状优化实现定制化弹性性能。 | [论文](https://doi.org/10.1016/j.addma.2022.103258) · [代码](https://github.com/latticeverse/ParametricShellLattice) · [引用](docs/citations/psl.bib) |
| 2023 · [AM](https://www.sciencedirect.com/journal/additive-manufacturing) | **TPMS-like shell lattices** | 基于周期边界和类极小曲面构造的参数化壳晶格族，将可探索的性能空间扩展到经典 TPMS 解析公式之外。 | [论文](https://doi.org/10.1016/j.addma.2023.103779) · [代码](https://github.com/latticeverse/TPMS-Like) · [引用](docs/citations/tpms-like.bib) |
| 2025 · [AM](https://www.sciencedirect.com/journal/additive-manufacturing) | **SPPM** | 使用 Wang 立方体规则和高斯核生成可制造的随机周期多孔微结构，在随机性和周期连通性之间取得平衡。 | [论文](https://doi.org/10.1016/j.addma.2025.104739) · [引用](docs/citations/sppm.bib) · 计划集成 |

## 2. 晶格数据集

数据集层将几何与仿真、学习和优化所需的标签及元数据连接起来。

| 数据集节点 | 在流程中的作用 | 输入 | 输出 / 资源 |
|:--|:--|:--|:--|
| **Dataset Construction** | 将生成的晶胞转换为标准化、带版本信息的样本。 | 几何参数、晶胞基向量、材料和工艺元数据。 | 几何资源、样本清单、溯源信息、数据划分元数据；计划集成数据模式（schema）。 |
| **Training and Benchmarks** | 为生成器和求解器建立可比较的训练集与评估集划分。 | 标准化样本、求解器标签、目标属性和固定随机种子。 | 训练/验证/测试清单、基准指标、校验和及评估配置。 |

这一层接收几何建模产生的样本，并向物理仿真、评估和三个优化分支提供输入。通过验证的候选结构会沿着 `Dataset expansion & optimization` 回到数据集，并保留物理场、属性、优化目标、约束和制造状态。数据集节点属于流程组件，而不是独立论文；使用这些样本时应引用生成样本的上游几何和求解器论文。

## 3. 物理仿真与评估

仿真负责生成物理场和等效属性。评估负责检查数值求解器、学习型代理模型和生成候选结构是否遵守相同的物理与数值约定。

| 年份 / 期刊或会议 | 项目 | 项目贡献 | 资源 |
|:--|:--|:--|:--|
| 2021 · [C&G](https://www.sciencedirect.com/journal/computers-and-graphics) | **Asymptotic Homogenization / Mechanical Property Profiles (AH / MPP)** | 确定性参考层，在明确的边界条件和材料约定下计算局部场、等效弹性属性、方向响应、强度相关性能曲线和最不利应力。 | [论文](https://doi.org/10.1016/j.cag.2021.07.021) · [代码](https://github.com/latticeverse/AsymptoticHomogenization) · [引用](docs/citations/ah-mpp.bib) |
| 2022 · [AM](https://www.sciencedirect.com/journal/additive-manufacturing) | **PH-Net** | 无需标签训练的 3D CNN，为一般平行六面体晶胞预测微观位移场，并由此得到局部属性和均匀化后的等效属性。 | [论文](https://doi.org/10.1016/j.addma.2022.103237) · [代码](https://github.com/latticeverse/phnet) · [引用](docs/citations/ph-net.bib) |
| 2025 · [arXiv](https://arxiv.org/abs/2506.17087) | **SLASH** | 受预条件共轭梯度方法启发的稀疏、周期性神经求解器，采用多级结构，在高分辨率下实现物理一致的均匀化。 | [论文](https://arxiv.org/abs/2506.17087) · [引用](docs/citations/slash.bib) · 计划集成 |
| 2026 · [SIGGRAPH](https://s2026.siggraph.org/) | **GMT** | 几何多重网格 Transformer，将稀疏点 Transformer 模块与多重网格层级结合，用于高保真的弹性和热均匀化。 | [论文](https://arxiv.org/abs/2604.26518) · [代码](https://github.com/latticeverse/GMT) · [引用](docs/citations/gmt.bib) |

对于每一个求解器，评估记录都应包含单位、坐标约定、张量排列顺序、边界条件、离散化方式、求解器容差、物理场误差和等效属性误差。只有在相同的评估规范（evaluation contract）下验证其性能结论后，生成的候选结构才能被接受。

## 4. 生成与优化

下游分支使用目标、约束和已经评估的样本搜索设计空间。每个分支都记录目标规格、随机种子、来源样本、候选几何、优化目标值和验证结果。

### 4.1 属性驱动的逆向设计

| 年份 / 期刊或会议 | 项目 | 项目贡献 | 资源 |
|:--|:--|:--|:--|
| 2025 · [SIGGRAPH](https://s2025.siggraph.org/) | **MIND** | 基于 Holoplane 表示的、具备对称性感知能力的潜空间扩散模型，联合编码晶格几何和物理响应，为目标性能生成候选结构。 | [论文](https://doi.org/10.1145/3721238.3730682) · [代码](https://github.com/latticeverse/MIND) · [引用](docs/citations/mind.bib) |
| 2026 · [ICML](https://icml.cc/Conferences/2026) | **AutoMS** | 多智能体神经符号系统，将面向仿真的进化搜索与语义任务分解结合，用于跨物理场逆向微结构设计。 | [论文](https://arxiv.org/abs/2603.27195) · [代码](https://github.com/latticeverse/AutoMS) · [引用](docs/citations/automs.bib) |

输入是一个目标属性，或一组耦合的物理目标。输出是一组具有多样性的候选晶格，以及预测属性、溯源信息和提交给仿真层的验证请求。

### 4.2 面向应用的优化

| 年份 / 期刊或会议 | 项目 | 项目贡献 | 资源 |
|:--|:--|:--|:--|
| 2025 · [C&S](https://www.sciencedirect.com/journal/computers-and-structures) | **Energy-absorbing PPL** | 面向应用的流程，将非线性仿真、MLP 代理模型和 NSGA-II 结合，在比吸能和峰值压溃力之间进行权衡。 | [论文](https://doi.org/10.1016/j.compstruc.2025.107880) · [引用](docs/citations/energy-absorbing-ppl.bib) · 计划集成 |
| 2025 · [M&D](https://www.sciencedirect.com/journal/materials-and-design) | **PETL (Joint-Enhanced Truss Lattice)** | 参数化的节点增强策略，在桁架交汇处附近重新分配材料，以降低应力集中，并在应用工况下提升刚度和强度。 | [论文](https://doi.org/10.1016/j.matdes.2025.113969) · [引用](docs/citations/petl.bib) · [代码](https://github.com/latticeverse/JointEnhancedTrussLattice) |

这一分支从应用目标和高保真仿真协议开始。代理模型或 Pareto 搜索提出候选结构，非线性仿真和物理测试决定哪些候选结构可以进入已验证数据集。

### 4.3 制造优化

| 年份 / 期刊或会议 | 项目 | 项目贡献 | 资源 |
|:--|:--|:--|:--|
| 2026 · [JCAD](https://www.jcad.cn/)| **MAPLE** | 面向制造约束的逆均匀化流程，将可微悬垂、封闭腔体和粉末去除约束纳入优化，并通过渐进式 Pareto 前沿构建保留可行候选结构。 | [论文](https://www.jcad.cn/article/doi/10.3724/SP.J.1089.2026-00157) · [引用](docs/citations/mo-ihd.bib) · 计划集成 |

输出是一组可制造的 Pareto 候选集，其中同时包含物理目标和制造可行性记录，可继续进行网格导出、制造和实验验证。制造约束属于优化规格的一部分，而不是最后阶段的修补步骤。

## 引用

流程图中各项目的引用文件维护在 [`docs/citations/`](docs/citations/) 中。对于科学结论，请使用论文条目；对于特定的软件发布版本，请使用软件或数据集条目。引用索引会标记尚未公开完整书目信息的项目。

使用任何组件时，请引用对应的论文或软件发布版本。若要引用 LatticeVerse 总仓库，请同时给出仓库地址以及所使用的提交或发布标签。

## 许可证

本仓库中的原创文档、数据模式（schema）、配置和集成代码采用 [MIT License](LICENSE)。链接项目的当前状态见 [`docs/THIRD_PARTY_NOTICES.md`](docs/THIRD_PARTY_NOTICES.md)。

MIT 许可证仅适用于本仓库中发布的内容。链接的独立项目、论文、数据集、模型权重、项目标志（logo）和第三方资源继续遵循各自的许可证及出版条款。目前部分链接项目使用 CC BY-NC 4.0，部分项目尚未公开代码许可证。README 中的链接不代表授予复制、修改或再分发这些材料的权限；在将其纳入其他发布包或再分发版本之前，请检查对应上游项目的许可条件。

LatticeVerse 标志是项目标志，根仓库许可证不授予商标权。如果本仓库未来发布原创数据集、图表或其他非代码材料，应在其发布元数据中单独声明数据或媒体许可证。代码许可证不会自动覆盖这些材料。
