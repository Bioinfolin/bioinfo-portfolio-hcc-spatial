# 技术决策与踩坑经验（作品集核心价值）

> 这部分是项目中最能体现"真实项目经验"的内容——每个条目都来自实际运行中解决的问题。

## 1. 空间转录组 h5ad 读取坑（rhdf5 直读 vs SeuratDisk）

**问题**：SAW 流程产出的 `.h5ad` 文件，用 SeuratDisk 转换读取时：
- 读到的层不对（把 log1p 层当原始 counts）
- 基因名错误/翻倍（`var/real_gene_name` 是 categorical 类型，直接 unlist 整个 group 会导致基因翻倍）

**解决**：用 `rhdf5` 直接读底层 HDF5 结构：
- `raw/X` = 原始 counts（uint32 CSR 稀疏矩阵）
- `X` = log1p 归一化后表达
- `var/_index` = Ensembl ID；`var/real_gene_name` = gene symbol
- categorical 列必须**分读 categories + codes** 再映射，不能整体 unlist

**教训**：工具链的"便捷接口"不代表数据正确，关键数据结构要亲自验证。

## 2. qs2 序列化断点体系

- 原因：`saveRDS` 对几十 GB 的中型对象太慢，且压缩率低
- 方案：统一 `qs2::qs_save()` / `qs2::qs_read()`（nthreads=32），扩展名保留 `.RDS`
- 收益：长计算（40 分钟级）中间结果秒级读写，支持断点续跑、快速迭代
- 注意：团队里如果有人用旧 `readRDS()` 会报错——需要用 `tryCatch` 兼容包装

## 3. AddModuleScore 的 CV 统计陷阱

**问题**：AddModuleScore 输出的均值≈0 时，变异系数 CV = sd/mean 会失真（负值分母、数值爆炸）

**解决**：空间异质性改用 **sd 与 IQR** 作为稳健统计量（对零基线鲁棒）

**应用**：免疫细胞毒性/耗竭模块的空间分布评分，用 sd/IQR 刻画"空间异质性"而不是"相对丰度"。

## 4. 空间解析度选择（bin50）

- Stereo-seq DNB 间距 0.5μm，bin50 = 25μm 分辨率
- 25μm 接近单细胞-多细胞尺度，是"生态位分析"的最佳入口（既有细胞结构信息，又保留生态区室关系）
- 更细的 bin20（10μm）用于确认关键结构的空间细节

## 5. 拷贝数变异验证（CopyKAT）

- 用途：肿瘤微环境中区分**恶性细胞与正常细胞**（CAF 注释前祛除恶性混淆）
- 输出：`bin_by_cell` 热图 + 细胞级预测分类（aneuploid vs diploid）
- 与 inferCNV 交叉验证，保证 CAF 亚群的"基质身份"干净

## 6. 配色与可视化规范

- 离散配色统一 `ggsci::scale_color_npg()` / `scale_fill_npg()`（10 色）
- 分类 ≤10 直接 npg；≤16 用 celltype_colors 扩展；>16 用 pal_jco/pal_lancet 补充
- **禁止 colorRampPalette 插值**（产生相近色，图内难以区分）
- 连续型用 gsea / viridis
- 同一变量同一色盘，项目周期内不随意变更——保证多图对比一致

## 7. 免疫细胞 A 候选判定（多方法加权）

- 候选：NK / CD4+ T / CD8+ T
- 判定依据：pos_ratio + AUCell Spearman + 加权综合得分
- 最终裁定进入模块 03 免疫功能评分验证（多证据链，不单一方法定论）

## 8. 日志与断点续跑规范

- 每模块：`run.log`（重定向 stdout/stderr）+ `progress_*.log`（分阶段标记）
- 轻量统计步骤不设断点（秒级重算，保证与最新逻辑一致）
- 只有分钟级以上的核心计算才设断点