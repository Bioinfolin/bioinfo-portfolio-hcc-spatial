# HCC 单细胞 + 空间转录组联合分析（作品集项目）

> **作品集说明**：本仓库展示基于华大 Stereo-seq 空间转录组与单细胞 RNA 测序的肝癌（HCC）肿瘤微环境分析完整技术路线。全部内容已经**脱敏**：不含客户原始数据、样本编号与保密信息，仅保留方法流程、技术决策与工程经验，可作为同类肿瘤微环境项目（单细胞 + 空间转录组联合分析）的通用模板。

## 📌 技术路线总览

```
单细胞(scRNA-seq) ──┬──► 免疫微环境解析（生态位定位/功能评分）
                    │
空间转录组(Stereo-seq, bin50) ─┴──► CAF 亚群注释 ──► CAF 演化轨迹
                                        │
                                        ▼
                              CellChat 细胞通讯（锁定信号轴）
                                        │
                                        ▼
                              TCGA/ICGC/GEO 队列预后特征验证
```

## 🧬 七个分析模块

| 模块 | 内容 | 核心方法 |
|---|---|---|
| 01 | 空间转录组 QC 与预处理 | 华大 SAW 上游定量（gef→bin50 h5ad）、rhdf5 直读 |
| 02 | 免疫细胞生态位定位 | ssGSEA / AddModuleScore / AUCell / SPOTlight |
| 03 | 免疫功能差异分析 | 细胞毒性 / 耗竭 / 迁移 / 活化 / 增殖模块评分 |
| 04 | CAF 亚群注释 | 肿瘤成纤维细胞二次聚类 + CopyKAT（拷贝数变异验证） |
| 05 | CAF 演化轨迹 | Monocle3 / Slingshot / PAGA / scVelo 多方法交叉 |
| 06 | 细胞通讯分析 | CellChat 锁定 CAF 亚群 ↔ 免疫细胞信号轴，回投空间验证共定位 |
| 07 | CAF 特征预后验证 | TCGA-LIHC / ICGC / GEO 队列生存分析与特征评分 |

## 🛠 技术栈

- **语言**：R 4.5.3（主）、Python（辅助，scVelo 等）
- **单细胞分析**：Seurat
- **空间转录组**：Stereo-seq（华大 SAW 上游），rhdf5 底层读取
- **通讯推断**：CellChat
- **轨迹推断**：Monocle3 / Slingshot / scVelo
- **序列化**：qs2（高性能 RDS 序列化，32 线程并行）
- **可视化**：ggplot2 + ggsci（npg 离散配色规范）

## 📁 目录结构

```
├── README.md                 # 本文件
├── docs/
│   ├── pipeline-overview.md  # 管线设计文档（数据流、执行顺序、断点机制）
│   └── technical-notes.md    # 技术决策与踩坑经验（作品集核心价值）
├── analysis_modules/
│   ├── 01_ST_QC.md           # 空转 QC 模块说明
│   ├── 02_Immune_Niche.md    # 免疫生态位模块说明
│   ├── 03_Immune_Function.md # 免疫功能模块说明
│   ├── 04_CAF_Annotation.md  # CAF 注释模块说明
│   ├── 05_CAF_Trajectory.md  # CAF 轨迹模块说明
│   ├── 06_CellChat.md        # 细胞通讯模块说明
│   └── 07_CAF_Signature.md   # 预后特征模块说明
└── .gitignore                # 数据/输出/断点排除规则
```

## 🔑 核心工程经验（摘要）

1. **空间数据读取**：SAW 产出的 h5ad 用 `rhdf5` 直接读取底层 HDF5（`raw/X` 为原始 counts、`X` 为 log1p），**避免 SeuratDisk 的层错误与基因名翻倍问题**；`var/real_gene_name` 为 categorical 类型需分开读 categories+codes
2. **断点机制**：长计算（40 分钟级）用 qs2 断点存盘（`qs_save`/`qs_read`，nthreads=32），图输出放在核心计算断点外，样式迭代无需重算
3. **统计口径**：AddModuleScore 均值≈0 时 CV（sd/mean）失真，空间异质性改用 **sd 与 IQR**（对零基线鲁棒）
4. **分辨率决策**：分析入口 bin50（25μm），备选 bin20（10μm）与 cellbin（细胞级）
5. **随机种子统一**：全流程固定随机种子，保证可复现

详见 [docs/technical-notes.md](docs/technical-notes.md)。

## ⚠️ 保密与脱敏说明

本仓库为求职作品集，**不含任何客户数据**。原始数据（Stereo-seq h5ad、scRNA-seq 矩阵）、分析输出（checkpoints、plots）均不包含在本仓库中，`.gitignore` 已配置对应排除规则。真实项目中此管线已完整运行于肝癌（HCC）肿瘤微环境课题，产出免疫细胞生态位定位、CAF 亚群注释与候选信号轴等关键结论。