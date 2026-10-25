## Experiment 1 results: numeric standardization

The only experimental change was standardizing POS, AF_EXAC, AF_TGP, AF_ESP, and Primary_Gene_Freq.
The StandardScaler was fitted on 944,604 training rows only. Both runs used the same 236,151 validation rows,
the same 506,039 test rows, initial weights, seed 42, minibatch order, and original FFN settings.
Each model was trained for the predetermined 20 epochs. The earlier full-training baseline remains a separate reference.

| Test metric | Before: unscaled | After: standardized | Change |
|---|---:|---:|---:|
| Accuracy | 67.8764% | 85.4071% | +17.5307 pp |
| Macro F1 | 0.202170 | 0.714268 | +0.512098 |
| Balanced accuracy | 25.0004% | 68.7372% | +43.7368 pp |
| Benign recall | 0.0016% | 62.7116% | +62.7101 pp |
| Likely benign recall | 100.0000% | 96.9882% | -3.0118 pp |
| Likely pathogenic recall | 0.0000% | 38.3847% | +38.3847 pp |
| Pathogenic recall | 0.0000% | 76.8642% | +76.8642 pp |

Standardization is an input preprocessing change, not an FFN architecture change.
These results describe one seed and one fixed split. Full-dataset one-hot and gene-frequency encodings
were inherited from the reproduced workflow, so this is a controlled scaling comparison within that workflow,
not a fully leakage-free evaluation. Future parameter selection should use validation data rather than repeated test comparisons.
