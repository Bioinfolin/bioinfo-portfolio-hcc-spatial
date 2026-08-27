# 模块 01：空间转录组 QC 与预处理

## 目标
对 Stereo-seq bin50 空转数据进行质量控制和基础预处理，建立分析入口对象。

## 输入
- SAW 上游产出：`*.bin50_1.0.h5ad`（raw/X = 原始 counts，X = log1p）
- 读取方式：`rhdf5` 直读底层 HDF5（避开 SeuratDisk 层错误和基因名翻倍问题）

## 关键步骤
1. rhdf5 读取 + 对象构建（稀疏矩阵、坐标 `obsm/spatial`、基因符号映射）
2. 质控指标：nGene / MID（分子计数）分布评估、低质量 bin 过滤
3. 标准化 + 高变基因 + PCA + UMAP 聚类（参考空间聚类合并图、分面图）
4. 16-cluster marker 表达分析，结合经典 marker（EPCAM/KRT19、PTPRC/CD3D/CD8A、DCN/LUM/COL1A1、PECAM1 等）初步注释组织结构

## 输出
- QC 后的分析对象（qs2 断点）
- 聚类图、质控图（plots/）

## 关键决策
- 分辨率：bin50 = 25μm 为主入口（详见 pipeline-overview.md）
- 组织区域先划分（肿瘤区/间质区/正常区），供下游生态位分析做空间背景
