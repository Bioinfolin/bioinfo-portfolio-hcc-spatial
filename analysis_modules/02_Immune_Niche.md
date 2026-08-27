# 模块 02：免疫细胞生态位定位

## 目标
在空间转录组上定位免疫细胞（CD8+T、NK、CD4+T 等）与 CAF 的生态位（niche），检验"免疫抑制性生态位"假说的空间基础。

## 关键步骤
1. 免疫细胞 A 候选判定：pos_ratio + AUCell Spearman + 加权综合得分（NK 初步候选，最终在模块 03 裁定）
2. 空间生态位表征：ssGSEA / AddModuleScore / AUCell 免疫细胞评分
3. 细胞类型空间共定位：SPOTlight / 空间富集分析
4. 生态位聚类与注释：识别肿瘤巢、免疫浸润区、间质屏障区

## 输出
- 免疫细胞空间评分图、生态位聚类图
- 候选免疫细胞 A 判定表（`02.immuneA_decision.csv`）

## 技术要点
- AddModuleScore 均值≈0 时空间异质性用 sd/IQR（CV 失真问题，见 technical-notes.md）
- 每一切片独立处理后再联合比较，避免批次效应污染空间结构
