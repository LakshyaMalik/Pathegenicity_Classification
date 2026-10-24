# Additional models: methods and results

## Data and preprocessing

The ClinVar dataset contains 1,686,794 variants. Preprocessing follows data7703.ipynb.
The group INFO extraction, first-gene/first-consequence rules, label mappings, zero imputation and eight-feature scheme were reused. Categories and gene frequencies were fitted on training rows.
The encoded matrix has 56 columns. Fitting, calibration, validation and test sizes are 944,604, 118,075, 118,076 and 506,039. Both tasks use the same test records.
The binary target merges benign/likely benign and pathogenic/likely pathogenic. No VUS records or cancer-specific filtering are represented.

## Models and selection

AdaBoost used 60 depth-2 trees and learning rate 0.5. LinearSVC used sparse training input, training-fitted scaling without centering, balanced class weights and C in {0.1, 1}. LightGBM used up to 350 trees, 31 leaves and learning rate 0.08 with 25-round validation early stopping.
AdaBoost and LightGBM compared standard and balanced sample weights. All candidates used separate sigmoid probability calibration. Validation macro-F1 selected variants; pathogenic-group recall broke ties. Test data were not used for selection.
Random Forest and XGBoost were retrained using the group model settings on these same partitions, with the same calibration procedure.
Soft voting compared the three additional models (SoftVoting_new) and the validation-best three of all five model families (SoftVoting). Each pool used equal averaging and quarter-step validation-selected weights; at least two weights were positive.

## Four Class results

| Model | Accuracy | Macro-F1 | Weighted F1 | Pathogenic recall | Pathogenic precision | False negatives |
|---|---:|---:|---:|---:|---:|---:|
| AdaBoost | 0.8345 | 0.6560 | 0.8100 | 0.7518 | 0.9884 | 24,735 |
| LinearSVC | 0.8283 | 0.6507 | 0.8056 | 0.7701 | 0.9791 | 22,913 |
| LightGBM | 0.8663 | 0.7398 | 0.8600 | 0.9174 | 0.9480 | 8,233 |
| RandomForest | 0.8554 | 0.7240 | 0.8533 | 0.9389 | 0.9572 | 6,090 |
| XGBoost | 0.8538 | 0.7420 | 0.8562 | 0.9539 | 0.8935 | 4,595 |
| SoftVoting_new_equal | 0.8440 | 0.6793 | 0.8244 | 0.7847 | 0.9843 | 21,457 |
| SoftVoting_new_weighted | 0.8633 | 0.7313 | 0.8549 | 0.8961 | 0.9617 | 10,355 |
| SoftVoting_equal | 0.8722 | 0.7549 | 0.8689 | 0.9372 | 0.9598 | 6,262 |
| SoftVoting_weighted | 0.8695 | 0.7546 | 0.8677 | 0.9440 | 0.9392 | 5,581 |

Selected variants: {'AdaBoost': 'standard', 'LinearSVC': 'C=1, balanced', 'LightGBM': 'standard'}.
Weighted-vote weights: {'SoftVoting_new': {'AdaBoost': 0.0, 'LinearSVC': 0.25, 'LightGBM': 0.75}, 'SoftVoting': {'XGBoost': 0.5, 'LightGBM': 0.25, 'RandomForest': 0.25}}.
Four-class pathogenic recall groups the two pathogenic labels; exact-class errors remain in macro-F1.

## Binary results

| Model | Accuracy | Macro-F1 | Weighted F1 | Pathogenic recall | Pathogenic precision | False negatives |
|---|---:|---:|---:|---:|---:|---:|
| AdaBoost | 0.9638 | 0.9433 | 0.9639 | 0.9203 | 0.8983 | 7,945 |
| LinearSVC | 0.9514 | 0.9199 | 0.9503 | 0.8224 | 0.9225 | 17,701 |
| LightGBM | 0.9738 | 0.9588 | 0.9738 | 0.9408 | 0.9271 | 5,901 |
| RandomForest | 0.9796 | 0.9676 | 0.9796 | 0.9415 | 0.9544 | 5,835 |
| XGBoost | 0.9709 | 0.9550 | 0.9712 | 0.9555 | 0.9024 | 4,436 |
| SoftVoting_new_equal | 0.9721 | 0.9558 | 0.9721 | 0.9247 | 0.9331 | 7,504 |
| SoftVoting_new_weighted | 0.9737 | 0.9585 | 0.9737 | 0.9378 | 0.9292 | 6,202 |
| SoftVoting_equal | 0.9796 | 0.9677 | 0.9796 | 0.9465 | 0.9497 | 5,328 |
| SoftVoting_weighted | 0.9815 | 0.9704 | 0.9814 | 0.9404 | 0.9647 | 5,938 |

Selected variants: {'AdaBoost': 'balanced', 'LinearSVC': 'C=1, balanced', 'LightGBM': 'balanced'}.
Weighted-vote weights: {'SoftVoting_new': {'AdaBoost': 0.0, 'LinearSVC': 0.25, 'LightGBM': 0.75}, 'SoftVoting': {'RandomForest': 0.5, 'LightGBM': 0.5, 'XGBoost': 0.0}}.


## Binary threshold comparison

| Rule | Threshold | Macro-F1 | Recall | Precision | False negatives |
|---|---:|---:|---:|---:|---:|
| default | 0.50 | 0.9704 | 0.9404 | 0.9647 | 5,938 |
| macro-F1 | 0.47 | 0.9706 | 0.9432 | 0.9623 | 5,663 |
| recall-priority | 0.24 | 0.9586 | 0.9684 | 0.9021 | 3,150 |

The recall-priority rule maximized validation recall with validation precision at least 0.90. Its test results describe the observed trade-off; the precision constraint is not guaranteed in future data.

## Error analysis

Error-analysis model for four_class: SoftVoting_equal (selected by validation macro-F1).
- missense_variant: 3,980 missed / 20,319 pathogenic-group records (19.59%).
- intron_variant: 1,336 missed / 1,341 pathogenic-group records (99.63%).
- non-coding_transcript_variant: 220 missed / 287 pathogenic-group records (76.66%).
Error-analysis model for binary: SoftVoting_weighted (selected by validation macro-F1).
- missense_variant: 3,656 missed / 20,319 pathogenic-group records (17.99%).
- intron_variant: 1,306 missed / 1,341 pathogenic-group records (97.39%).
- synonymous_variant: 187 missed / 187 pathogenic-group records (100.00%).

Confidence-only four-class errors were separated from pathogenic-to-benign errors. Subgroup tables and 20 false-negative examples per task are included.

## Unseen primary genes

The additional binary LightGBM test contained 524,339 variants from 5,035 primary genes absent from fitting, calibration and validation. Unknown-gene rows excluded: 339.
Macro-F1 was 0.9179; pathogenic recall was 0.7858; precision was 0.9635.
Standard weighting and fixed LightGBM hyperparameters were used for this separate experiment. The main random-split binary LightGBM selected balanced weighting. The cohort, class mix and weighting differ, so this is a sensitivity test rather than a direct estimate of the effect of holding out genes. Secondary genes are not isolated.

## Files and environment

All new training and analysis code is in hao_models.ipynb. Figures are in figures/. data/prepared.joblib stores first-stage data, features and partitions. data/experiment.joblib stores selected additional models, the gene-holdout model and evaluation probabilities. Running all cells reuses saved fits; REFIT_MODELS=True repeats training. DATA specifies the source CSV path.
Python 3.12.14; numpy 2.5.4; pandas 3.1.0; scipy 1.18.1; scikit-learn 1.9.1; xgboost 3.4.1; lightgbm 4.7.0; matplotlib 3.11.2; joblib 1.6.0; ipykernel 7.4.0.
The supplied file was obtained from the shared Google Drive link. Its upstream release date and license remain unverified.
