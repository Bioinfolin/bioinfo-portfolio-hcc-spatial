# 管线设计文档

## 1. 数据流

```
原始数据
 ├─ 空间转录组：Stereo-seq basecalling → SAW 上游定量（gef） → bin50_1.0.h5ad
 │    （bin50 = 25μm 分辨率，DNB 间距 0.5μm × 50）
 └─ 单细胞：scRNA-seq 矩阵

分析主线
 01 空转 QC ──► 02 免疫生态位 ──► 03 免疫功能评分
                                    │
 单细胞全谱注释 ◄──── (并行分支) ────┤
 ──► 04 CAF 二次聚类+注释（CopyKAT 验证）──► 05 CAF 轨迹
 ──► 06 CellChat 通讯（锁定 CAF↔免疫信号轴，回投空转验证共定位）
 ──► 07 队列预后验证（TCGA-LIHC / ICGC / GEO）
```

## 2. 执行策略

| 原则 | 说明 |
|---|---|
| 双线并行 | 空转链（01→03）与单细胞链（04→05）并行，06 汇合，07 收尾 |
| 断点驱动 | 每个长计算模块（>分钟级）设置 checkpoints/ 断点（qs2 格式） |
| 图算分离 | 样式频繁迭代的图输出放在核心计算断点之外 |
| 种子统一 | 全流程固定随机种子 20260814，保证可复现 |
| 日志追踪 | 每模块输出 run.log + progress 日志，支持断点续跑 |

## 3. 关键分辨率决策（空间转录组）

| 分辨率 | 物理尺寸 | 用途 |
|---|---|---|
| bin50_1.0.h5ad | 25 μm | **分析主入口**（QC、生态位、模块评分） |
| bin20_1.0.h5ad | 10 μm | 备选（更精细的空间结构） |
| cellbin | 细胞级 | 单细胞分辨率验证 |

## 4. 断点格式约定

- 所有中间结果统一 **qs2 序列化**（`qs2::qs_save` / `qs2::qs_read`，nthreads=32）
- 扩展名保留 `.RDS`，但 **不能用 base `readRDS()` 读取**（会报 "unknown input format"）
- 外部读取须 `library(qs2); qs_read(path, nthreads=32)`
- 兼容老数据：`tryCatch(qs2::qs_read(path), error = function(e) readRDS(path))`