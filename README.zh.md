# eve-lexs-nsp5

**基于Learn–Expand–Sample的SARS-CoV-2高活性NSP5 (Mpro)底物设计流程。**

<p align="center"><a href="README.md">English</a> | 中文</p>

---

## 概述

本仓库包含论文[`Li, Lin, et al. "Viral Protease-Initiated Lytic Cell Death as a Universal Antiviral mRNA Therapy." Cell, 2026.`](https://www.cell.com/cell/abstract/S0092-8674(26)00751-8)中**AI-driven accelerated evolution generates VIDAs with enhanced antiviral potency**部分的计算流程与相关数据。

VIPA (Viral protease-Initiated Pyroptosis Activator)是一种通用抗病毒mRNA疗法平台，其核心机制是将gasdermin D的天然切割位点替换为目标病毒蛋白酶的底物序列，使VIPA仅在病毒感染细胞中被激活并触发细胞焦亡，从而选择性清除感染细胞。其中，在应用到SARS-CoV-2的尝试中，为增强SARS-CoV-2 VIPA的切割效率与突变耐受性，我们开发了一套基于进化约束底物设计框架——**Learn–Expand–Sample (LEXS)**——对NSP5 (Mpro)蛋白酶的8氨基酸切割基序(P6–P2′)进行从头设计。

### 生物安全声明

为最大限度降低潜在生物安全风险，用于候选序列生成与采样的脚本暂未公开发布。但可根据合理请求向符合资质的研究人员提供，供其用于研究目的，并需经过生物安全审查。

### 流程

|     阶段     | 描述                                                                                                                                                     |
|:----------:|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Learn**  | 训练[EVE](https://github.com/OATML-Markslab/EVE) (Evolutionary model of Variant Effects)模型，从NCBI Virus数据库收集的261条非重复天然SARS-CoV-2 NSP5 P6–P2′ 切割基序中学习进化约束。 |
| **Expand** | 用训练好的EVE模型生成100,000条候选序列，并通过进化评分量化每条序列的适应度；生成集覆盖了天然序列空间并向未观测区域扩展。                                                                                      |
| **Sample** | 对生成序列进行去冗余与评分过滤后，通过K-Means聚类和质心近端采样选出30条多样性代表候选序列，进行后续实验验证。                                                                                            |

<p align="center">
  <img src="assets/workflow.svg" alt="流程示意图" width="600">
</p>


### 关键结果

- 在独立测试集（25条已知切割结果的序列）上，EVE进化评分的分类性能：**ROC-AUC = 0.80，F1 = 0.78**
- 25条成功合成的候选肽中，**7条**切割速率超过野生型底物**2倍**以上
- 其中3条VIPA变体在SARS-CoV-2感染细胞中表现出 **>90 %** 的病毒抑制率

<table>
  <tr>
    <td align="center"><img src="assets/cleavage.svg" alt="AI生成肽候选物的切割动力学" width="600"></td>
    <td align="center"><img src="assets/antiviral.svg" alt="HeLa-ACE2细胞中的抗病毒活性" width="200"></td>
  </tr>
  <tr>
    <td align="center"> AI生成候选肽的切割动力学</td>
    <td align="center"> 细胞实验测得的抗病毒活性</td>
  </tr>
</table>

---

## 目录结构

```
├── src/
│   ├── README.md                      # EVE准备说明
│   └── EVE/                           # [未包含] 需自行准备
│
├── data/
│   ├── raw/
│   │   ├── SARS-CoV-2_nsp_sites.xlsx  # 原始 NSP5 P6–P2′ 序列（来自NCBI）
│   │   └── nsp5_testset.fasta         # 测试集（ID中包含标签）
│   └── processed/
│       ├── nsp5_short.fasta
│       ├── nsp5_testset_8aa.fasta
│       ├── sars_nsp5_site_8aa_mapping.csv
│       └── nsp5_testset_8aa_mapping.csv
│
├── checkpoints/
│   ├── logistic_model.pkl             # 训练好的逻辑回归模型权重
│   └── sars_nsp5_8aa/
│       ├── nsp5_sars_nsp5_8aa_final   # VAE模型权重
│       └── weights/
│           └── nsp5_theta_0.01.npy    # VAE的theta权重
│
├── notebooks/
│   ├── 1_Data.ipynb                   # 数据准备
│   ├── 2_Model.ipynb                  # VAE训练与测试
│   └── 3_Generate.ipynb               # 序列生成（暂不公开）
│
├── logs/
│   └── sars_nsp5_8aa/
│       └── nsp5_sars_nsp5_8aa_losses.csv  # VAE训练损失日志
│
└── results/
    ├── EVE/
    │   └── sars_nsp_5_8aa_testset/
    │       ├── evol_indices/
    │       │   └── nsp5_20000_samples.csv  # EVE计算的测试集点突变的进化指数
    │       └── mutations/
    │           └── nsp5_all_singles.csv     # 测试集所有点突变列表
    └── generated_seq/
        └── sars_nsp5_8aa_final_selected.csv  # 30条最终候选序列
```

---

## 环境配置

### 依赖

- Python 3.10+
- Biopython
- PyTorch（EVE所需）
- scikit-learn、NumPy、SciPy、pandas
- matplotlib、logomaker

### 克隆EVE

EVE源代码(`commit 460d70efeeeded58bc69227a203540d68953ae88`)未包含在此仓库中，需克隆到`src/`下：

```bash
git clone https://github.com/OATML-Markslab/EVE.git src/EVE
```

---

## Notebooks

### 1 — 数据准备 (`notebooks/1_Data.ipynb`)

- 加载从NCBI获取的NSP5 P6–P2′序列
- 可视化训练集的序列氨基酸分布
- 生成处理后的FASTA文件用于训练和测试
- 生成EVE需要的的映射CSV
- 检查训练集与测试集是否有重叠

### 2 — 模型训练与评估 (`notebooks/2_Model.ipynb`)

- 在训练集上训练EVE（调用`train_VAE.py`）
- 用训练好的EVE模型对所有测试集序列评分
- 在测试集上评估分类性能（ROC曲线）
- 训练逻辑回归分类器，建立EVE分数与活性的关系
- 保存模型至 `checkpoints/logistic_model.pkl`

---

## 引用

如果本仓库对您的研究有帮助，请引用：

> Li, Lin, et al. "Viral protease-Initiated Pyroptosis Activator mRNA therapy as a Universal Antiviral Strategy." Cell, 2026.

---

## 许可

本仓库及其自定义代码（不含外部开发的EVE框架）采用**Apache License 2.0**分发。详见[LICENSE](LICENSE)文件。

---

## 致谢

- [EVE (OATML-Markslab)](https://github.com/OATML-Markslab/EVE)—变分自编码器框架
- [NCBI Virus](https://www.ncbi.nlm.nih.gov/labs/virus/)—NSP5切割基序数据来源
