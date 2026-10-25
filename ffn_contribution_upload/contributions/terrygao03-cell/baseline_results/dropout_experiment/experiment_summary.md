## Experiment 3 results: Dropout 0.3 to 0.2

Only the Dropout probability changed. Both runs used standardized inputs, hidden width 128, the same saved scaler,
944,604 training rows, 236,151 validation rows, initial weights, seed 42, and minibatch order.
Adam lr=0.001, batch size=32, and the predetermined 20 epochs remained unchanged.
The saved Dropout=0.3 model is the control. No additional test evaluation was performed.

| Validation metric | Dropout 0.3 | Dropout 0.2 | Change |
|---|---:|---:|---:|
| Accuracy | 85.4373% | 85.5135% | +0.0762 pp |
| Macro F1 | 0.718025 | 0.718276 | +0.000251 |
| Balanced accuracy | 69.4142% | 69.3459% | -0.0684 pp |
| Benign recall | 62.1265% | 61.9426% | -0.1840 pp |
| Likely benign recall | 96.6642% | 96.8226% | +0.1585 pp |
| Likely pathogenic recall | 41.6744% | 40.8396% | -0.8347 pp |
| Pathogenic recall | 77.1918% | 77.7786% | +0.5868 pp |
| Training seconds | 313.7 | 326.9 | +13.2 |

Final-epoch validation macro F1 favors Dropout=0.2 in this run.
The macro F1 change is +0.000251. One seed and one validation split cannot establish
whether a small difference is stable. The inherited full-dataset categorical and gene-frequency encoding limitation remains.

The experiment overview now records the unscaled control and all three modifications using validation metrics only.
For reporting, describe the substantial scaling improvement separately from smaller architecture/regularization effects.
Further confirmation can repeat promising configurations across several seeds using the same evaluation protocol.
Reserve test evaluation for the final selected configuration.
