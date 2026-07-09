# Interpretable Multi-Omics Ovarian Cancer Survival Prediction

This repository presents an interpretable machine-learning project for **epithelial ovarian cancer survival prediction** using three processed omics layers: **mRNA-seq**, **DNA methylation**, and **log2 copy-number alteration (log2CNA)**. The goal is not only to classify survival outcome, but also to identify the molecular features that repeatedly influenced the model's decisions.

The project is **research-oriented** and should be interpreted as a computational and biological exploration. It is **not** a clinically validated diagnostic or prognostic tool.

---

## Executive Summary

The final workflow trains separate **L1-regularized Logistic Regression** models for CNA, methylation, and mRNA-seq data. L1 regularization was selected because it supports sparse, gene-level interpretability by shrinking weaker coefficients toward zero.

The three omics-specific models are then combined using a **probability-weighted Logistic Regression ensemble**. Instead of relying on one omics layer, the ensemble combines the predicted probabilities from the CNA, methylation, and mRNA-seq models using validation-performance-based weights.

Explainability is performed using SHAP-based feature attribution. For each test-sample prediction, the strongest local explanation features are extracted and aggregated across omics layers. The final interpretation in this README focuses only on the **combined repeated XAI features occurring at least 5 times** across the full multi-omics workflow.

---

## Dataset and Multi-Omics Inputs

The project uses three processed omics layers:

| Omics layer | Patients before overlap filtering | Features | Feature meaning |
|---|---:|---:|---|
| DNA methylation | 418 | 22,600 | CpG/methylation sites linked to gene regulation |
| mRNA-seq | 218 | 19,076 | Gene-expression features |
| log2CNA | 417 | 25,128 | Gene-level copy-number alteration features |

Patient overlap across omics layers is limited. The workflow aligns samples across CNA, methylation, and mRNA-seq before model training and ensemble prediction.

---

## Survival Threshold Selection

The survival target was converted into a binary label using month-level survival thresholds. The 42-month threshold was selected because it produced the smallest survived-vs-deceased class difference among the tested thresholds.

<p align="center">
  <img src="results/figures/01_survival_threshold_class_balance.png" alt="Survival threshold class balance" width="700">
</p>

**Figure 1. Survival-threshold class-balance analysis.**  
The class difference was lowest around **42 months**, making it the most balanced cutoff for the main survival-classification experiment.

---

## Exploratory CNA Literature-Gene Overlay

A t-SNE visualization was generated for normalized log2CNA features, and literature-derived ovarian cancer gene panels were overlaid on the broader CNA feature space.

<p align="center">
  <img src="results/figures/07_log2cna_tsne_literature_gene_overlay.png" alt="log2CNA t-SNE literature gene overlay" width="700">
</p>

**Figure 2. Normalized t-SNE of log2CNA data with literature-derived epithelial ovarian cancer gene panels overlaid.**  
Grey points represent all CNA genes/features. Colored points represent genes from literature-derived ovarian cancer panels. This plot is used only as an exploratory sanity check showing that literature-supported genes are present in the processed CNA feature space. It should **not** be interpreted as proof of survival causality or biological clustering.

---

## Modeling Workflow

The workflow follows this structure:

1. Load processed mRNA-seq, methylation, and log2CNA datasets.
2. Align patients across datasets using sample IDs.
3. Remove samples without usable survival labels.
4. Select the 42-month survival threshold for binary classification.
5. Split aligned samples into train and test sets.
6. Normalize features.
7. Remove low-variance features.
8. Rank remaining features using correlation with the survival label.
9. Add back genes from literature-derived ovarian cancer panels where relevant.
10. Train separate tuned Logistic Regression models for CNA, methylation, and mRNA-seq.
11. Evaluate each model using accuracy, balanced accuracy, precision, recall, F1-score, and confusion matrix.
12. Combine model probabilities using validation-performance-based ensemble weights.
13. Use SHAP explanations to identify repeatedly important features.
14. Aggregate recurrent features across the three omics layers.
15. Interpret only the final combined features occurring **5+ times**.

---

## Model Performance

### CNA-only Logistic Regression

<p align="center">
  <img src="results/figures/02_confusion_matrix_cna_lr.png" alt="CNA LR confusion matrix" width="400">
</p>

| Model | Accuracy | Balanced Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| CNA LR | 0.7170 | 0.7165 | 0.7143 | 0.7407 | 0.7273 |

The CNA model produced the strongest individual-omics test performance in this run.

---

### Methylation-only Logistic Regression

<p align="center">
  <img src="results/figures/03_confusion_matrix_methylation_lr.png" alt="Methylation LR confusion matrix" width="400">
</p>

| Model | Accuracy | Balanced Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Methylation LR | 0.6604 | 0.6603 | 0.6667 | 0.6667 | 0.6667 |

The methylation model showed moderate survival-classification signal and contributed to the final probability-weighted ensemble.

---

### mRNA-seq-only Logistic Regression

<p align="center">
  <img src="results/figures/04_confusion_matrix_mrnaseq_lr.png" alt="mRNA-seq LR confusion matrix" width="400">
</p>

| Model | Accuracy | Balanced Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| mRNA-seq LR | 0.6415 | 0.6403 | 0.6333 | 0.7037 | 0.6667 |

The mRNA-seq model had lower overall accuracy than CNA, but retained meaningful recall and contributed a strong transcriptomic explanation signal through CCL14.

---

### Probability-Weighted Ensemble Model

<p align="center">
  <img src="results/figures/05_confusion_matrix_weighted_ensemble_lr.png" alt="Probability-weighted ensemble confusion matrix" width="400">
</p>

| Model | Accuracy | Balanced Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Probability-weighted LR ensemble | **0.7547** | **0.7543** | **0.7500** | **0.7778** | **0.7636** |

The probability-weighted ensemble produced the strongest overall result in this run, improving over the individual CNA, methylation, and mRNA-seq models.

Ensemble weights from the current run:

| Omics model | Ensemble weight |
|---|---:|
| CNA | 0.3039 |
| Methylation | 0.3431 |
| mRNA-seq | 0.3530 |

---

## Final Combined Explainable AI Results

The final XAI interpretation focuses only on the **combined repeated features occurring at least 5 times** across the methylation, mRNA-seq, and CNA explanation dictionaries.

<p align="center">
  <img src="results/figures/06_top_xai_genes_frequency.png" alt="Combined top XAI genes frequency" width="1000">
</p>

**Figure 7. Combined features occurring 5+ times across all models.**  
Frequency means how often a gene or feature appeared among the strongest local explanation signals. It does **not** prove that the gene causes survival differences.

### Combined XAI Feature Frequency Table

| Feature / Gene | Methylation | mRNA-seq | CNA | Combined frequency |
|---|---:|---:|---:|---:|
| FRMD5 | 0 | 0 | 13 | **13** |
| CCL14 | 0 | 13 | 0 | **13** |
| LRRN1 | 0 | 0 | 11 | **11** |
| AADAC | 3 | 0 | 7 | **10** |
| RIPPLY2 | 0 | 0 | 8 | **8** |
| ATP5A1 | 0 | 0 | 8 | **8** |
| RAD54B | 6 | 0 | 1 | **7** |
| NUAK2 | 0 | 0 | 7 | **7** |
| FOXJ1 | 0 | 0 | 7 | **7** |
| CAT | 0 | 0 | 7 | **7** |
| GUSB | 0 | 0 | 6 | **6** |
| RFT1 | 0 | 0 | 6 | **6** |
| MRAP2 | 0 | 0 | 6 | **6** |
| NF1 | 0 | 0 | 6 | **6** |
| C10orf4 | 5 | 0 | 0 | **5** |
| KLK10 | 5 | 0 | 0 | **5** |
| TGM7 | 0 | 0 | 5 | **5** |
| GRB7 | 1 | 0 | 4 | **5** |
| IGF2 | 4 | 0 | 1 | **5** |

---

## Biological Interpretation of Final Combined XAI Features

Important interpretation note: these are **model-derived explanation features**, not proven causal biomarkers. A feature can appear because it captures expression signal, methylation regulation, copy-number dosage, or a correlated genomic region. CNA features especially should be interpreted carefully because they may reflect broader copy-number structure rather than direct gene-expression activity.

| Feature / Gene | Model signal | Evidence level | Literature-supported relevance | Simple explanation | Reference(s) used |
|---|---:|---|---|---|---|
| **FRMD5** | 13; CNA | Exploratory; process-supported | FRMD5 is annotated as a FERM-domain gene and has experimental evidence linking it to cell motility and cell-matrix adhesion. Direct epithelial ovarian cancer evidence is limited, but ovarian cancer spread is strongly connected to extracellular matrix interaction, adhesion, invasion, and peritoneal dissemination. | This is like a gene connected to how cells stick, move, and hold their shape. If copy-number changes affect this region, the model may be detecting signals related to tumor cells becoming more mobile or invasive. Do not call it a confirmed ovarian biomarker yet. | [30], [31], [42], [43] |
| **CCL14** | 13; mRNA-seq | Direct EOC evidence | CCL14 is a chemokine involved in immune signalling. A published epithelial ovarian cancer study reported CCL14 as a potential prognostic biomarker and independent prognostic factor, with higher expression associated with more favourable prognosis. | This gene is part of immune communication. If its expression is higher, it may reflect a tumor environment where immune-related signalling is different, which can affect survival patterns. | [6] |
| **LRRN1** | 11; CNA | Exploratory; process-supported | LRRN1 is a leucine-rich-repeat gene with cell-surface/adhesion-related biology. Direct ovarian-cancer-specific evidence is limited, so this is interpreted as a recurrent CNA signal rather than a validated ovarian cancer gene. Ovarian cancer literature supports adhesion and extracellular-matrix pathways as relevant to invasion and spread. | Think of this as a cell-surface communication or adhesion-related signal. The model may be seeing copy-number structure around a region that affects how tumor cells interact with their surroundings. | [32], [42], [43] |
| **AADAC** | 10; methylation + CNA | Direct ovarian cancer evidence | AADAC expression signatures have been associated with ovarian cancer prognosis and immune-cell infiltration. | This gene may connect metabolism-like activity with immune patterns in the tumor. In simple terms, its repeated appearance suggests the model may be detecting a survival-related biological environment, not just a random feature. | [7] |
| **RIPPLY2** | 8; CNA | Exploratory | RIPPLY2 is officially annotated as a ripply transcriptional repressor involved in vertebrate somitogenesis and transcriptional repression. Direct ovarian cancer evidence is not well established, so this should be treated as a model-derived CNA signal only. | Developmental regulators help control body-patterning and cell-state programs. When such regions appear in cancer data, they may suggest abnormal regulation, but this needs validation before making a strong claim. | [44] |
| **ATP5A1** | 8; CNA | Cancer-process supported | ATP5A1 encodes a subunit of mitochondrial ATP synthase. ATP synthase and mitochondrial energy metabolism have been discussed as cancer-relevant, and mitochondrial biology is important in ovarian cancer progression and therapy response. | Cancer cells need energy to grow, divide, repair damage, and resist stress. A repeated ATP5A1-linked CNA signal may reflect energy-metabolism differences in tumors. | [22], [23] |
| **RAD54B** | 7; methylation + CNA | Direct ovarian cancer evidence | RAD54B participates in homologous recombination DNA repair. RAD54B disruption has been reported to impair homologous recombination repair and increase ovarian cancer cell sensitivity to PARP inhibition. | This is one of the strongest biology links. If DNA repair genes are altered, cancer cells may accumulate damage. In ovarian cancer, DNA-repair weakness is important because it affects tumor behavior and PARP-inhibitor response. | [8] |
| **NUAK2** | 7; CNA | Ovarian-cancer evidence; still emerging | Ovarian cancer work reported through AACR has described NUAK2 as highly overexpressed in ovarian cancer, with nuclear NUAK2 associated with tumor aggressiveness and poorer survival. | NUAK2 is a kinase, meaning it helps switch signalling pathways on or off. If overactive, it may help cancer cells survive stress, move, or behave more aggressively. | [20], [21] |
| **FOXJ1** | 7; CNA | Direct HGSOC survival evidence | A high-grade serous ovarian carcinoma study found that increased FOXJ1 protein expression was associated with improved overall survival. | FOXJ1 is linked to ciliated-cell biology. In this context, higher FOXJ1 may mark a tumor state associated with better survival rather than aggressive growth. | [9] |
| **CAT** | 7; CNA | Cancer-process supported | CAT encodes catalase, an antioxidant enzyme. Oxidative stress and antioxidant defence are important in ovarian cancer biology, and catalase has been discussed as a cancer-relevant antioxidant defence target. | Tumors produce and experience chemical stress. Catalase helps break down harmful peroxide molecules. If this pathway changes, cancer cells may survive stress and treatment pressure better. | [24], [25] |
| **GUSB** | 6; CNA | Ovarian profiling caution | GUSB has been evaluated as a stable reference gene for normalization in human serous ovarian cancer gene-expression studies. Because of that, its recurrence should be interpreted cautiously rather than overclaimed as a cancer driver. | This is a caution gene. Since GUSB can behave like a housekeeping/reference gene, its recurrence should not be overclaimed as a cancer driver. It may reflect stable background signal or copy-number context. | [14] |
| **RFT1** | 6; CNA | Exploratory; pathway-supported | RFT1 is involved in N-linked glycosylation through lipid-linked oligosaccharide translocation. Direct RFT1-specific ovarian cancer evidence is limited, but ovarian cancer literature supports glycosylation changes as relevant to membrane proteins, signalling, adhesion, immune interaction, and biomarker biology. | Glycosylation is like adding sugar tags to proteins. These tags can change how cancer cells signal, stick, hide from immune cells, or spread. RFT1 is therefore biologically plausible but not directly validated here. | [33], [34], [26], [27] |
| **MRAP2** | 6; CNA | Exploratory; pathway-supported | MRAP2 regulates melanocortin/GPCR-related signalling. Direct MRAP2-specific ovarian cancer evidence is limited, but GPCR/endocrine receptor signalling has been reviewed as relevant to ovarian cancer progression and metastasis. | GPCR signalling is one way cells respond to outside signals. A recurrent MRAP2-linked CNA signal may point to survival-associated genomic structure, but this should stay exploratory. | [35], [45] |
| **NF1** | 6; CNA | Direct ovarian/HGSOC genomic evidence | NF1 is a tumor suppressor and negative regulator of RAS signalling. TCGA's integrated ovarian carcinoma analysis and ovarian serous carcinoma studies identify NF1 alterations in ovarian carcinoma. | NF1 normally acts like a brake on growth signalling. If this brake is weakened, RAS-related growth pathways can become more active, supporting uncontrolled cell division when combined with other cancer errors. | [4], [15] |
| **C10orf4 / ANTKMT** | 5; methylation | Exploratory; process-supported | C10orf4, also known as ANTKMT/FAM173A, is linked to mitochondrial lysine methylation and mitochondrial respiration. Direct ovarian cancer evidence is limited, but mitochondrial biology is relevant to ovarian cancer. | This methylation feature may point to mitochondrial regulation. Since cancer cells often change energy use, the signal is plausible but should not be called a confirmed ovarian cancer gene. | [36], [37], [23] |
| **KLK10** | 5; methylation | Direct ovarian cancer evidence | KLK10 is a kallikrein-family gene. KLK10 expression and methylation have been studied in ovarian tumor diagnosis and prognosis. | KLK10 is connected to secreted/protease biology. Changes in methylation may affect gene regulation, and kallikrein-family signals have been studied as ovarian cancer biomarkers. | [12], [13] |
| **TGM7** | 5; CNA | Exploratory; pathway-supported | TGM7 belongs to the transglutaminase family. Direct TGM7-specific ovarian evidence is limited, but tissue transglutaminase biology has been linked to ovarian cancer cell survival signalling, metastasis, adhesion, and chemotherapy resistance. | Transglutaminases help crosslink proteins and influence tissue structure. In ovarian cancer, related enzymes can help tumor cells stick to surfaces and resist cell death. TGM7 itself should remain exploratory. | [38], [28], [29] |
| **GRB7** | 5; methylation + CNA | Direct ovarian cancer evidence | GRB7 is an adaptor protein linked to growth-factor receptor signalling and is located near ERBB2/HER2. Ovarian cancer studies have reported GRB7/GRB7v involvement in ovarian carcinogenesis, GRB7/ERK/FOXM1 signalling in ovarian cancer cell aggressiveness, and GRB7 amplification/protein expression in ovarian cancer. | GRB7 helps transmit growth signals inside cells. If this region is amplified or dysregulated, it may help cancer cells receive stronger growth and survival messages. | [17], [18], [19], [39], [40] |
| **IGF2** | 5; methylation + CNA | Direct ovarian cancer evidence | IGF2 belongs to the insulin-like growth factor signalling axis. Epithelial ovarian cancer studies have linked IGF2 overexpression with altered methylation in the IGF2/H19 imprinting region. | IGF2 acts like a growth signal. If methylation changes increase IGF2 activity, cells may receive stronger pro-growth and survival messages, which can contribute to tumor development when combined with other mutations. | [10], [11] |
---

## Main Biological Themes from the Final Combined Features

### 1. Copy-number-driven structural and survival signals

Many of the strongest combined features came from the CNA model, including **FRMD5, LRRN1, RIPPLY2, ATP5A1, NUAK2, FOXJ1, CAT, RFT1, GUSB, MRAP2, NF1, TGM7, GRB7, and IGF2**. This suggests that copy-number alteration carried a strong recurrent explanatory signal in this run.

This fits the biology of high-grade serous ovarian cancer, which is known for widespread genomic instability and copy-number alteration.

### 2. Immune signalling and prognosis

**CCL14** was the dominant mRNA-seq feature. Its epithelial ovarian cancer prognostic literature makes it one of the most biologically interpretable transcriptomic signals in this project.

### 3. DNA repair and PARP-relevant biology

**RAD54B** is highly relevant because homologous recombination repair and PARP-inhibitor response are central themes in ovarian cancer biology. RAD54B recurrence in methylation-linked explanations is therefore biologically meaningful.

### 4. Mitochondrial metabolism and oxidative stress

**ATP5A1, CAT, and C10orf4/ANTKMT** point toward energy metabolism, oxidative stress, and mitochondrial regulation. These are plausible survival-related pathways because cancer cells must adapt energy production and stress defence to keep growing.

### 5. Growth-factor signalling and uncontrolled division

**NF1, GRB7, and IGF2** connect the model output to growth signalling. In simple terms, these genes relate to whether cells receive, transmit, or suppress growth messages. If growth signals become too strong or growth brakes fail, cells can divide more than they should, and with other DNA errors this can contribute to cancer formation and progression.

### 6. Adhesion, extracellular matrix, and spread

**FRMD5, LRRN1, RFT1, and TGM7** connect to adhesion, glycosylation, extracellular matrix, or tissue-structure biology. These are relevant because epithelial ovarian cancer often spreads through the peritoneal cavity, where adhesion and cell-surface interactions matter.

---

## Interpretation Limits

These explainability results should be interpreted as **model-level recurrence patterns**, not as proof of causality. A high frequency means the feature repeatedly influenced the trained model's predictions. It does not prove that changing that gene would change patient survival.

Before any biological claim is treated as strong, these features would need:

- external cohort validation,
- statistical testing beyond SHAP recurrence,
- survival modelling such as Cox regression or Kaplan-Meier analysis,
- gene-expression/methylation/CNA directionality analysis,
- and ideally experimental validation.

---

## Key Takeaway

The strongest result of this project is the combination of:

1. a probability-weighted multi-omics ensemble with the strongest predictive performance, and  
2. a compact, interpretable set of recurrent model-explanation features.

The most important current XAI features are:

```text
FRMD5, CCL14, LRRN1, AADAC, RIPPLY2, ATP5A1, RAD54B, NUAK2, FOXJ1, CAT,
GUSB, RFT1, MRAP2, NF1, C10orf4/ANTKMT, KLK10, TGM7, GRB7, IGF2
```

Among these, **CCL14, AADAC, RAD54B, FOXJ1, NF1, KLK10, GRB7, and IGF2** have stronger ovarian-cancer-specific support. **FRMD5, LRRN1, RIPPLY2, RFT1, MRAP2, C10orf4/ANTKMT, and TGM7** should be framed more cautiously as exploratory model-derived signals or pathway-supported candidates.

---

## References

[1] Millstein J, Budden T, Goode EL, et al. **Prognostic gene expression signature for high-grade serous ovarian cancer.** *Annals of Oncology.* 2020. PubMed: https://pubmed.ncbi.nlm.nih.gov/32473302/

[2] Srinivasamurthy BC, Ramamoorthi C. **The Progression and Prospects of the Gene Expression Profiling in Ovarian Epithelial Cancer.** *Gynecology and Minimally Invasive Therapy.* 2024. PubMed: https://pubmed.ncbi.nlm.nih.gov/39184260/

[3] Lundberg SM, Lee S-I. **A Unified Approach to Interpreting Model Predictions.** *NeurIPS.* 2017. https://arxiv.org/abs/1705.07874

[4] Cancer Genome Atlas Research Network. **Integrated genomic analyses of ovarian carcinoma.** *Nature.* 2011;474:609–615. PubMed: https://pubmed.ncbi.nlm.nih.gov/21720365/

[5] Arora T, Mullangi S, Lekkala MR. **Epithelial Ovarian Cancer.** *StatPearls / NCBI Bookshelf.* PubMed: https://pubmed.ncbi.nlm.nih.gov/33620837/

[6] Cai Y, et al. **C-C motif chemokine 14 as a novel potential biomarker for predicting the prognosis of epithelial ovarian cancer.** PubMed: https://pubmed.ncbi.nlm.nih.gov/32218842/

[7] Feng J, et al. **Signature of arylacetamide deacetylase expression is associated with prognosis and immune infiltration in ovarian cancer.** PubMed: https://pubmed.ncbi.nlm.nih.gov/34902961/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC8784941/

[8] Liu P, et al. **RAD54B mutations enhance the sensitivity of ovarian cancer cells to PARP inhibitors.** PubMed: https://pubmed.ncbi.nlm.nih.gov/35952757/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC9463535/

[9] Weir A, et al. **Increased FOXJ1 protein expression is associated with improved overall survival in high-grade serous ovarian carcinoma.** PubMed: https://pubmed.ncbi.nlm.nih.gov/36323878/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC9814937/

[10] Murphy SK, et al. **Frequent IGF2/H19 domain epigenetic alterations and elevated IGF2 expression in epithelial ovarian cancer.** PubMed: https://pubmed.ncbi.nlm.nih.gov/16603642/

[11] Huang Z, et al. **Increased intragenic IGF2 methylation is associated with repression of insulator activity and elevated expression in serous epithelial ovarian carcinoma.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC3662894/

[12] El Sherbini MA, et al. **KLK10 exon 3 unmethylated PCR product concentration in ovarian cancer diagnosis and prognosis.** PubMed: https://pubmed.ncbi.nlm.nih.gov/29690914/

[13] Shvartsman HS, et al. **Overexpression of kallikrein 10 in epithelial ovarian carcinomas.** PubMed: https://pubmed.ncbi.nlm.nih.gov/12821340/

[14] Li YL, et al. **Identification of suitable reference genes for gene expression studies of human serous ovarian cancer.** PubMed: https://pubmed.ncbi.nlm.nih.gov/19622337/

[15] Sangha N, et al. **Neurofibromin 1 (NF1) defects are common in human ovarian serous carcinomas and co-occur with TP53 mutations.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC2586687/

[16] Farley J, et al. **Associations between ERBB2 amplification and progression-free survival and overall survival in advanced stage, suboptimally-resected epithelial ovarian cancers.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC6944288/

[17] Wang Y, et al. **Differential functions of growth factor receptor-bound protein 7 and its variant in ovarian carcinogenesis.** PubMed: https://pubmed.ncbi.nlm.nih.gov/20388850/

[18] Chan DW, et al. **Targeting GRB7/ERK/FOXM1 signaling pathway impairs aggressiveness of ovarian cancer cells.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC3527599/

[19] Zeng M, et al. **Grb7 gene amplification and protein expression by FISH and IHC in ovarian cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC4637669/

[20] Xu C, et al. **NUAK2 is highly overexpressed in ovarian cancer and the overexpression of its nuclear form correlates with tumor aggressiveness.** *Cancer Research* AACR abstract. https://aacrjournals.org/cancerres/article/80/16_Supplement/5906/644304/Abstract-5906-NUAK2-is-highly-overexpressed-in

[21] Xu C, et al. **NUAK2: The key player in ovarian cancer progression.** *Cancer Research* AACR abstract. https://aacrjournals.org/cancerres/article/85/8_Supplement_1/6721/760384

[22] Wang T, et al. **Defueling the cancer: ATP synthase as an emerging target in cancer therapy.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC8517097/

[23] Shukla P, et al. **The mitochondrial landscape of ovarian cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC8163040/

[24] Ding DN, et al. **Insights into the Role of Oxidative Stress in Ovarian Cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC8516553/

[25] Glorieux C, et al. **Targeting catalase in cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC11539659/

[26] Anugraham M, et al. **Specific glycosylation of membrane proteins in epithelial ovarian cancer cell lines.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC4159645/

[27] Wanyama FM, et al. **Glycomic-Based Biomarkers for Ovarian Cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC8065431/

[28] Cao L, et al. **Tissue transglutaminase protects epithelial ovarian cancer cells from cisplatin-induced apoptosis by promoting cell survival signaling.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC2556973/

[29] Yakubov B, et al. **Extracellular tissue transglutaminase activates noncanonical NF-κB signaling and promotes metastasis in ovarian cancer.** PubMed: https://pubmed.ncbi.nlm.nih.gov/23730209/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC3664993/

[30] NCBI Gene. **FRMD5 FERM domain containing 5.** https://www.ncbi.nlm.nih.gov/gene/84978

[31] Hu J, et al. **FERM domain-containing protein FRMD5 regulates cell motility and cell-matrix adhesion.** PubMed: https://pubmed.ncbi.nlm.nih.gov/25448675/

[32] NCBI Gene. **LRRN1 leucine rich repeat neuronal 1.** https://www.ncbi.nlm.nih.gov/gene/57633

[33] NCBI Gene. **RFT1 glycolipid translocator homolog.** https://www.ncbi.nlm.nih.gov/gene/91869

[34] Chen S, et al. **Rft1 catalyzes lipid-linked oligosaccharide translocation across the endoplasmic reticulum membrane.** *Nature Communications.* 2024. https://www.nature.com/articles/s41467-024-48999-3

[35] NCBI Gene. **MRAP2 melanocortin 2 receptor accessory protein 2.** https://www.ncbi.nlm.nih.gov/gene/112609

[36] NCBI Gene. **ANTKMT / FAM173A / C10orf4.** https://www.ncbi.nlm.nih.gov/gene/65990

[37] Małecki JM, et al. **Human FAM173A is a mitochondrial lysine-specific methyltransferase that targets adenine nucleotide translocase and affects mitochondrial respiration.** PubMed: https://pubmed.ncbi.nlm.nih.gov/31213526/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC6682728/

[38] NCBI Gene. **TGM7 transglutaminase 7.** https://www.ncbi.nlm.nih.gov/gene/116179

[39] NCBI Gene. **GRB7 growth factor receptor bound protein 7.** https://www.ncbi.nlm.nih.gov/gene/2886

[40] NCBI Gene. **ERBB2 erb-b2 receptor tyrosine kinase 2.** https://www.ncbi.nlm.nih.gov/gene/2064

[41] NCBI MedGen / MedlinePlus Genetics. **Ovarian cancer.** https://www.ncbi.nlm.nih.gov/medgen/216027

[42] Cho A, et al. **The Extracellular Matrix in Epithelial Ovarian Cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC4629462/

[43] Rafii A, et al. **High-prevalence and broad spectrum of cell adhesion pathway mutations in serous epithelial ovarian cancer.** PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC3492115/


[44] NCBI Gene. **RIPPLY2 ripply transcriptional repressor 2.** https://www.ncbi.nlm.nih.gov/gene/134701

[45] Zhang Q, Madden NE, Wong AST, Chow BKC, Lee LTO. **The Role of Endocrine G Protein-Coupled Receptors in Ovarian Cancer.** *Frontiers in Endocrinology.* 2017. PubMed: https://pubmed.ncbi.nlm.nih.gov/28439256/ ; PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC5383648/
