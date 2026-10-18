# Investigating Class Imbalance Handling Techniques for Machine Learning-Based Network Intrusion Detection

## Abstract

Class imbalance remains a significant challenge in machine learning-based network intrusion detection, particularly for identifying rare attack categories. This study investigates four imbalance-handling techniques using the UNSW-NB15 dataset: class weighting, random oversampling, random undersampling, and SMOTENC. Logistic Regression and Random Forest are employed as baseline classifiers, with imbalance-handling experiments focusing on Random Forest.

Five-fold Stratified Group Cross-Validation is conducted to reduce overlap between identical feature patterns across training and validation subsets. Experimental results show that class weighting achieves the highest mean Macro-F1 (0.6041) and Balanced Accuracy (0.6585), compared with 0.5616 and 0.5416 for the unweighted Random Forest baseline.

On the official test set, class weighting improves Macro-F1 from 0.4585 to 0.5010 and substantially increases recall for minority attack categories, including Backdoor and Worms. However, these improvements are accompanied by reduced overall accuracy and substantial false positives for certain categories.

Further analysis identifies duplicated feature patterns, conflicting labels, and train-test feature overlap. The findings highlight the importance of balancing minority-class detection performance, precision, and evaluation reliability in network intrusion detection research.

**Keywords:** Network Intrusion Detection, Class Imbalance, UNSW-NB15, Random Forest, SMOTENC, Machine Learning

## 1. Introduction

Machine learning-based network intrusion detection systems (NIDS) aim to distinguish malicious from legitimate network traffic. However, substantial class imbalance can make rare attack categories difficult to detect reliably.

The UNSW-NB15 benchmark contains 175,341 training records covering normal traffic and nine attack categories [1]. While the training set includes 56,000 Normal samples, only 130 belong to the Worms category, illustrating the severity of class imbalance.

This study evaluates four imbalance-handling techniques using Random Forest: class weighting, random oversampling, random undersampling, and SMOTENC. Performance is assessed using Macro-F1, Balanced Accuracy, and per-class Precision and Recall, with particular attention to minority-class detection and false-positive trade-offs.

Additionally, duplicated feature patterns, conflicting labels, and train-test feature overlap are investigated to assess potential limitations in the reliability of model evaluation.

## 2. Methodology

### 2.1 Dataset, Preprocessing and Models

The experiments use the official UNSW-NB15 training (175,341 records) and test (82,332 records) partitions, with 42 input features: 39 numerical and three categorical (`proto`, `service`, and `state`).

Numerical features undergo median imputation and standardisation; categorical features undergo most-frequent imputation and one-hot encoding. Preprocessing is fitted within training partitions to avoid validation leakage.

A binary Logistic Regression model and multiclass Logistic Regression and Random Forest models are implemented using scikit-learn [3]. Random Forest uses 100 trees and a fixed random seed of 42.

### 2.2 Imbalance-Handling Techniques

Four Random Forest configurations are compared against an unweighted baseline:

1. Class Weighting: Uses balanced class weights.
2. Random Oversampling: Duplicates minority samples to match the largest class.
3. Random Undersampling: Reduces Normal, Generic, and Exploits to 15,000 samples each.
4. SMOTENC: Increases Analysis, Backdoor, Shellcode, and Worms to 5,000 samples each using a categorical-aware extension of SMOTE [2].

Resampling methods are implemented using imbalanced-learn [4] and applied only to training data within each evaluation split.

### 2.3 Evaluation Protocol

Five-fold Stratified Group Cross-Validation is performed on the official training partition, grouping records by identical input feature patterns to prevent such patterns from crossing training-validation boundaries.

Macro-F1 and Balanced Accuracy guide model selection, supplemented by Accuracy and per-class Precision, Recall, and F1-score.

The selected class-weighted Random Forest is retrained on the full training partition and evaluated on the official test set. A separate analysis of test records without matching training feature patterns examines sensitivity to test-set composition.

<div style="page-break-before: always;"></div>

## 3. Results

### 3.1 Cross-Validation Performance

Table 1 summarises the mean performance across five grouped cross-validation folds.

**Table 1. Five-fold cross-validation results.**

| Method | Macro-F1 | Balanced Accuracy |
|--------|----------|-------------------|
| Baseline RF | 0.5616 | 0.5416 |
| Class Weight | **0.6041** | **0.6585** |
| Random Oversampling | 0.5883 | 0.6170 |
| Random Undersampling | 0.5967 | 0.6019 |
| SMOTENC | 0.5694 | 0.5991 |

Class weighting achieved the highest mean Macro-F1 and Balanced Accuracy, improving upon the baseline by 0.0425 and 0.1169, respectively.

### 3.2 Official Test Set Performance

**Table 2. Official test set performance.**

| Metric | Baseline RF | Weighted RF |
|--------|-------------|-------------|
| Accuracy | 0.7544 | 0.6921 |
| Macro-F1 | 0.4585 | **0.5010** |
| Balanced Accuracy | 0.4828 | **0.5871** |

The weighted model improved Macro-F1 and Balanced Accuracy at the expense of overall Accuracy. Backdoor recall increased from 0.0892 to 0.5763, Shellcode from 0.6667 to 0.8016, and Worms from 0.0682 to 0.5227. However, Backdoor precision was only 0.0502, indicating substantial false positives.

![Per-Class Recall Comparison](../results/figures/final_class_recall_comparison.png)

**Figure 1.** Per-class recall of baseline and class-weighted Random Forest models on the official test set.

### 3.3 Data Reliability Analysis

Feature-pattern analysis identified 8,541 official test records (10.37%) matching patterns in the training set. On the remaining 73,791 records, the weighted model achieved Macro-F1 of 0.5328 and Balanced Accuracy of 0.6208.

These differences suggest sensitivity to test-set composition but do not establish that overlapping patterns caused the performance changes.

<div style="page-break-before: always;"></div>

## 4. Discussion and Limitations

Class weighting achieved the strongest cross-validation performance but revealed important trade-offs on the official test set. Although minority-class recall improved, overall Accuracy decreased, and false positives remained substantial. The weighted model produced 6,351 false-positive Backdoor predictions and misclassified 11,442 Normal records as Fuzzers. Backdoor precision was only 0.0502, highlighting the potential burden of false alarms in practical intrusion detection.

Data quality also complicates evaluation. UNSW-NB15 contains duplicated feature patterns and conflicting labels for identical feature inputs. Grouped cross-validation prevents identical patterns from appearing across training-validation boundaries, but cannot eliminate other dependencies. Furthermore, 10.37% of official test records share feature patterns with the training set. Performance differences on the non-overlapping subset may reflect changes in data composition rather than the direct effect of overlap.

Several limitations remain. Experiments were restricted to one dataset, with only 44 Worms records in the official test set. Extensive hyperparameter optimisation, external validation, and statistical significance testing were not conducted. Additionally, the official test set had been inspected during earlier baseline experiments and was therefore not completely untouched before final model evaluation.

Consequently, the results support class weighting under the evaluated conditions but do not establish its universal superiority. Future work should investigate external datasets and methods that improve minority-class recall while controlling false positives.

## 5. Conclusion

This study compared four class imbalance-handling techniques for Random Forest-based network intrusion detection using UNSW-NB15. Class weighting achieved the highest mean Macro-F1 (0.6041) and Balanced Accuracy (0.6585) under five-fold grouped cross-validation.

On the official test set, class weighting improved Macro-F1 from 0.4585 to 0.5010 and Balanced Accuracy from 0.4828 to 0.5871, with substantial recall gains for rare attacks such as Backdoor and Worms. However, overall accuracy declined, and false positives remained problematic.

The findings demonstrate that improved minority-class detection must be evaluated alongside precision and data quality. Future work should examine external datasets and approaches that balance minority-class sensitivity with false-positive control.

## 6. References

[1] Moustafa, N., & Slay, J. (2015). UNSW-NB15: A comprehensive data set for network intrusion detection systems (UNSW-NB15 network data set). 2015 Military Communications and Information Systems Conference (MilCIS), 1-6.

https://doi.org/10.1109/MilCIS.2015.7348942

[2] Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic Minority Over-sampling Technique. Journal of Artificial Intelligence Research, 16, 321-357.

https://doi.org/10.1613/jair.953

[3] Pedregosa, F., et al. (2011). Scikit-learn: Machine Learning in Python. Journal of Machine Learning Research, 12, 2825-2830.

https://jmlr.org/papers/v12/pedregosa11a.html

[4] Lemaître, G., Nogueira, F., & Aridas, C. K. (2017). Imbalanced-learn: A Python Toolbox to Tackle the Curse of Imbalanced Datasets in Machine Learning. Journal of Machine Learning Research, 18(17), 1–5.

https://jmlr.org/papers/v18/16-365.html