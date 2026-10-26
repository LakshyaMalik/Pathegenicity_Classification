## Experiment 2 results: hidden width 64 to 128

Only hidden width changed. Both runs used standardized inputs, the same 944,604 training rows and 236,151 validation rows,
the saved Experiment 1 scaler, seed 42, minibatch order, and predetermined 20 epochs.
Dropout=0.3, Adam lr=0.001, batch size=32, and the four-class target remained unchanged.
The previously saved 64-unit model is the control. No additional test evaluation was performed.

| Validation metric | Hidden 64 | Hidden 128 | Change |
|---|---:|---:|---:|
| Accuracy | 85.3382% | 85.4373% | +0.0991 pp |
| Macro F1 | 0.712790 | 0.718025 | +0.005235 |
| Balanced accuracy | 68.6248% | 69.4142% | +0.7894 pp |
| Benign recall | 62.4161% | 62.1265% | -0.2896 pp |
| Likely benign recall | 96.9518% | 96.6642% | -0.2876 pp |
| Likely pathogenic recall | 38.1364% | 41.6744% | +3.5380 pp |
| Pathogenic recall | 76.9949% | 77.1918% | +0.1969 pp |
| Parameters | 3908 | 7812 | +3904 |
| Training seconds | 276.0 | 313.7 | +37.7 |

Final-epoch validation macro F1 favors hidden width 128 in this run.
The change in macro F1 is +0.005235. This is a single seed and one validation split;
a small difference does not establish a reliable advantage. Full-dataset one-hot and gene-frequency encodings remain
an inherited limitation. Use validation metrics for subsequent tuning and reserve test evaluation for the final selected configuration.

The next isolated experiment can change Dropout from 0.3 to 0.2 while keeping the selected width and all other settings fixed.
For reporting, record each change, validation macro F1, per-class recall, and training cost, including changes that do not improve results.
