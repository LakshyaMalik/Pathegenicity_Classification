# Interpretation of four-class XGBoost and binary LightGBM/Random Forest

## Scope and verification

This analysis explains four-class XGBoost, binary LightGBM, and binary Random Forest from the group's `hao_models.ipynb` experiment at upstream commit `bdb51d3b6fa87fc6688586e16917feb995f4addb`.

The original calibrated binary LightGBM was loaded from the group cache. The four-class XGBoost and binary Random Forest were independently refitted using the upstream settings because the group cache retained their predictions but not their fitted model objects. Both refits use the original fitting and separate calibration partitions. All three models agreed with their cached probabilities on the first 2,000 validation records within a maximum absolute difference of 2.98e-8. This is a numerical check on those records, not proof of identity on every input.

The prepared dataset contains 1,686,794 variants and 56 encoded columns. Fitting, calibration, validation and test partitions contain 944,604, 118,075, 118,076 and 506,039 records, respectively. The input caches and the earlier group/FFN notebooks were not modified.

## Methods

Global importance uses 6,000 validation records, sampled with stratification on the four-class labels. Eight original feature groups are permuted, with one-hot columns kept together. Each group is shuffled five times. The reported importance is the absolute decrease in macro-F1; error bars are the standard deviation across these five permutations. They are not confidence intervals over new datasets or retraining.

Local explanations enumerate all 256 coalitions of eight groups against 64 stratified fitting-background records. They are exact grouped empirical interventional Shapley values for this specified masking game, explaining each complete sigmoid-calibrated class probability. They are not Tree SHAP. Baseline probability plus all group contributions reconstructs each explained probability; the largest recorded residual is approximately 1.45e-15.

Four cases per model were selected deterministically from a common 10,000-record test pool: a correct benign-group prediction, a correct pathogenic-group prediction, a false negative, and a false positive. Exact class correctness is required for four-class correct cases. A four-class false negative means that the true label is in the pathogenic group but the argmax four-class label is in the benign group; it does not apply a threshold to the sum of the two pathogenic-class probabilities.

## Global findings

| Model and task | Sample baseline macro-F1 | Largest importance | Other notable importance |
|---|---:|---|---|
| XGBoost, four-class | 0.7334 | MC_Clean: 0.3201 | AF_TGP: 0.1328; Primary_Gene_Freq: 0.0324; AF_EXAC: 0.0313 |
| LightGBM, binary | 0.9559 | MC_Clean: 0.4285 | Primary_Gene_Freq: 0.0317; AF_TGP: 0.0307 |
| Random Forest, binary | 0.9707 | MC_Clean: 0.4302 | CLNVC: 0.0827; Primary_Gene_Freq: 0.0442; AF_TGP: 0.0435 |

These baseline scores describe the 6,000-record validation sample used for importance, not full-test performance. A drop of 0.3201 is an absolute macro-F1 difference, not the percentage of predictions caused by a feature.

**Shared reliance on molecular consequence.** `MC_Clean` is the dominant feature group for each selected model under this analysis. The models depend strongly on the retained molecular-consequence annotation. Importance alone does not say that every missense or nonsense annotation has the same direction of effect.

**Four-class XGBoost.** Population frequency `AF_TGP` is the second-largest group. The gene-frequency proxy and ExAC frequency provide smaller contributions to global predictive performance. Their close importance estimates should not be treated as a stable ordering.

**Binary model differences.** Both binary models use identical validation rows and permutation orders. Their reliance on `MC_Clean` is similar in magnitude. Random Forest relies much more on variant type `CLNVC` (0.0827 versus 0.0034 for LightGBM), and its score is also more sensitive to chromosome and position. This describes the reliance of these fitted models on this sample; it does not establish biological importance or superiority of an algorithm. LightGBM's secondary estimates for gene frequency and AF_TGP are close and their precise order is uncertain.

## Feature meanings

- `MC_Clean`: the first molecular-consequence term retained during upstream cleaning; additional listed consequences were discarded.
- `AF_TGP`, `AF_EXAC`, and `AF_ESP`: population allele-frequency fields. They can be correlated, and missing values were filled with zero.
- `Primary_Gene_Freq`: the number of fitting records assigned to the extracted primary gene. It is a dataset-count proxy, not gene expression or biological gene importance.
- `CLNVC`: variant type, such as single-nucleotide variant or deletion/indel.
- `CHROM` and `POS`: chromosome and genomic position. Reliance can reflect annotation, sampling or genomic context; it is not evidence of causality.

Gene names shown in case tables are metadata. The model receives the frequency proxy rather than an encoded gene identity.

## Local examples

**Correct predictions shared across models.** Row 63 is a synonymous variant correctly assigned to the benign group by all three models (Likely_benign in four-class XGBoost). MC_Clean contributes +0.2161 to the XGBoost Likely_benign output, +0.2006 to the LightGBM benign output, and +0.1417 to the Random Forest benign output. Row 2368 has a nonsense annotation and is correctly assigned to the pathogenic group by all models. MC_Clean contributes +0.1971 to XGBoost's Likely_pathogenic output and +0.7005/+0.6725 to the binary LightGBM/Random Forest pathogenic outputs. These are background-dependent contributions to different class outputs, not interchangeable numerical measures of biological effect.

**XGBoost false negative, row 24878.** The reference label is Likely_pathogenic, but the argmax prediction is Likely_benign (probability 0.4841 versus 0.4548 for Likely_pathogenic). AF_EXAC contributes +0.1605 to the Likely_benign output, whereas MC_Clean contributes -0.2669. For the true Likely_pathogenic output, the consequence contribution is positive and the AF_EXAC contribution is negative. The model retains pathogenic evidence but ultimately selects a benign-group class. The exact class probabilities, not just the sign of one contribution, determine the error.

**XGBoost false positive, row 2924.** A Likely_benign variant is predicted as Likely_pathogenic with probability 0.5257. MC_Clean contributes +0.3593 to that output, while POS contributes -0.0564. This shows a strong model association with the consequence group accompanying an incorrect prediction; it does not establish that the reference label is wrong.

**LightGBM false negative, row 13655.** A pathogenic-group variant is assigned benign probability 0.5948. Position (+0.0638) and gene frequency (+0.0405) push the benign output upward. Molecular consequence (-0.2217) and population-frequency fields oppose that output. The selected background's benign baseline is already 0.8103, so these opposing contributions can lower benign probability substantially while it remains above 0.5.

**LightGBM false positive, row 4392.** The model assigns pathogenic probability 0.5421 to a benign-group variant. MC_Clean (+0.2744), AF_TGP (+0.0508), and AF_EXAC (+0.0339) push the pathogenic output upward, partly offset by POS (-0.0459).

**Random Forest false negative, row 24878.** This is the same variant as the XGBoost false negative, with the reference label collapsed to the pathogenic group. Random Forest assigns benign probability 0.9605. AF_EXAC contributes +0.2763 to the benign output, outweighing opposing consequence (-0.0824), gene-frequency (-0.0227), and AF_TGP (-0.0134) contributions. This is an example of a high-confidence error: a large predicted probability does not guarantee a correct label.

**Random Forest false positive, row 15651.** A benign-group Indel is assigned pathogenic probability 0.8433. The largest positive contributions are MC_Clean (+0.2537), the gene-count proxy (+0.1852), and chromosome (+0.0864). All eight group contributions to this output are positive under the selected background.

The binary models' selected error cases differ, so these cases do not by themselves establish which model handles the same difficult variants better. Rows 63 and 2368 are common correct cases; row 24878 is shared by XGBoost and Random Forest. Other same-variant comparisons require additional predictions/explanations.

## Limitations and report wording

The analysis describes fitted-model reliance, not biological causality or clinically validated risk. Permutation does not provide a universal direction or frequency cutoff. Correlated fields can share importance, and interventional masking can create unusual feature combinations. Results use one global sample, five permutations, a small empirical background and four selected cases per model. The selected cases do not estimate error prevalence. Binary and four-class score-drop magnitudes should not be compared directly. No additional hyperparameter search or performance improvement is claimed by this interpretation step.

A suitable contribution statement is: "We interpreted four-class XGBoost and binary LightGBM/Random Forest using grouped permutation importance and local grouped Shapley explanations of calibrated predictions. Molecular consequence was the dominant feature group in all three models. The binary models differed in their reliance on variant type and genomic context. Correct, false-negative and false-positive examples showed how competing feature contributions can produce both accurate predictions and confident errors."

## Output guide

- Three `*_global_importance.png` figures and matching grouped-importance CSVs: main global results.
- `binary_importance_comparison.png`: comparison on common validation rows.
- Three `*_local_cases.png` figures: correct predictions, false negatives and false positives.
- `local_cases.csv` and `local_grouped_shap_values.csv`: case metadata and all class/group contributions.
- `analysis_record.json`, `model_provenance_checks.csv`, and `verification_report.json`: design and verification evidence.

## Method references

- https://scikit-learn.org/stable/modules/permutation_importance.html
- https://shap.readthedocs.io/en/latest/generated/shap.ExactExplainer.html
- https://arxiv.org/abs/1705.07874
