# Explanation of the selected FFN

## Selected configuration

Standardized numeric inputs; hidden width 128; Dropout 0.2; Adam lr=0.001; batch size 32; 20 epochs; seed 42. Validation accuracy=0.855135, macro F1=0.718276. This is the highest recorded validation macro F1 among tested configurations; its advantage over Dropout 0.3 is only about 0.000251 for one seed. No new model training or test evaluation was performed.

## Global feature importance

Grouped permutation importance on 30,000 stratified validation rows, 10 repeats; sample baseline macro F1=0.716570. Error bars show one permutation SD.

| Feature | Mean macro F1 decrease | SD |
|---|---:|---:|
| MC_Clean | 0.294299 | 0.001317 |
| AF_TGP | 0.137905 | 0.001593 |
| Primary_Gene_Freq | 0.036527 | 0.001648 |
| CHROM | 0.024150 | 0.001737 |
| AF_ESP | 0.017613 | 0.000908 |
| AF_EXAC | 0.012574 | 0.000770 |
| CLNVC | 0.011125 | 0.000788 |
| POS | 0.006733 | 0.000947 |

## Leading features for this FFN

1. **MC_Clean**: mean macro F1 decrease 0.2943. Text description of the first predicted molecular consequence retained from MC. The FFN uses consequence categories such as missense, synonymous, or frameshift. This is a model association; additional consequences were discarded during cleaning.

2. **AF_TGP**: mean macro F1 decrease 0.1379. Allele frequency reported from the 1000 Genomes Project. This supplies another population-frequency measurement. Correlation with the other frequency fields can distribute importance.

3. **Primary_Gene_Freq**: mean macro F1 decrease 0.0365. Number of dataset rows assigned to the extracted primary gene. This is a dataset-frequency proxy, not gene expression, gene identity, or biological gene importance. Full-dataset counting is an inherited evaluation limitation.

4. **CHROM**: mean macro F1 decrease 0.0242. Chromosome containing the variant. Reliance can reflect chromosome-specific annotation and sampling patterns; it is not a causal explanation.

5. **AF_ESP**: mean macro F1 decrease 0.0176. Allele frequency reported from GO-ESP. Interpret as dependence on this population-frequency field, not a universal pathogenicity threshold. Missing values were filled with zero.


## Local grouped SHAP examples

Exact enumeration over eight groups (256 coalitions) with 64 stratified training-background rows; all four class outputs are explained. Cases are the first validation rows matching the stated correct/error categories.

### Correct Benign (row 1002402)

True label: Benign; predicted label: Benign; model probability for the predicted class: 0.6234; probability for the true class: 0.6234. Observed MC_Clean=intron_variant, CLNVC=single_nucleotide_variant, primary gene=FGD4. The model uses gene frequency, not the gene name.

Largest positive contributions to the predicted class: AF_TGP (+0.5452), MC_Clean (+0.0105). Largest negative contributions: CLNVC (-0.0191), AF_ESP (-0.0150).

### Correct Pathogenic (row 1645645)

True label: Pathogenic; predicted label: Pathogenic; model probability for the predicted class: 0.7219; probability for the true class: 0.7219. Observed MC_Clean=nonsense, CLNVC=single_nucleotide_variant, primary gene=IL2RG. The model uses gene frequency, not the gene name.

Largest positive contributions to the predicted class: MC_Clean (+0.5606), CHROM (+0.0457). Largest negative contributions: CLNVC (-0.0110), POS (-0.0027).

### Misclassified Likely_pathogenic (row 663701)

True label: Likely_pathogenic; predicted label: Pathogenic; model probability for the predicted class: 0.6901; probability for the true class: 0.3054. Observed MC_Clean=frameshift_variant, CLNVC=Deletion, primary gene=KMT2E. The model uses gene frequency, not the gene name.

Largest positive contributions to the predicted class: MC_Clean (+0.5143), CLNVC (+0.0391). Largest negative contributions: Primary_Gene_Freq (-0.0041).

### Misclassified Pathogenic (row 1456437)

True label: Pathogenic; predicted label: Likely_pathogenic; model probability for the predicted class: 0.6170; probability for the true class: 0.2365. Observed MC_Clean=missense_variant, CLNVC=single_nucleotide_variant, primary gene=LDLR. The model uses gene frequency, not the gene name.

Largest positive contributions to the predicted class: MC_Clean (+0.2731), CHROM (+0.0976). No negative contributions to this predicted class under the selected background.

## Interpretation of the four local cases

- **Correct Benign (row 1002402):** AF_TGP=0.02117 contributes about +0.5452 to the Benign output relative to the training background. The model assigns Benign probability 0.6234 and the prediction matches the label. This shows dependence on the observed population-frequency value in this case, not a universal frequency cutoff.
- **Correct Pathogenic (row 1645645):** MC_Clean=nonsense contributes about +0.5606 to the Pathogenic output. The model assigns Pathogenic probability 0.7219 and matches the label. The consequence group is the dominant contributor for this prediction.
- **Likely_pathogenic predicted as Pathogenic (row 663701):** MC_Clean=frameshift_variant contributes about +0.5143 to the Pathogenic output. The model favors Pathogenic (0.6901) over the true Likely_pathogenic class (0.3054). This illustrates a failure to distinguish these adjacent labels despite a strong consequence signal; it does not establish that the reference label is wrong.
- **Pathogenic predicted as Likely_pathogenic (row 1456437):** The largest contributions to the Likely_pathogenic output come from MC_Clean=missense_variant and chromosome context, followed by the gene-count proxy. The predicted class has probability 0.6170, while the true Pathogenic class has probability 0.2365. The case shows that a plausible molecular-consequence association can still accompany a wrong four-class prediction.

These are explanations of model outputs under the stated reference background. The primary gene names are descriptive case metadata; the FFN uses the gene-count feature rather than gene identity.

## Limits

The ranking explains model reliance, not biological causality. Permutation gives no universal direction of effect. Population-frequency fields are correlated, and absent frequencies were filled with zero. Gene frequency is a dataset-count proxy learned from the full dataset in the inherited workflow; this prevents a strictly leakage-free interpretation. Local interventional masking can break feature correlations, uses only 64 reference rows, and depends on the chosen grouping. Softmax outputs are model probabilities, not calibrated clinical risks. One seed and a fixed split do not establish stable superiority of the selected hyperparameters.

## References

- https://scikit-learn.org/stable/modules/permutation_importance.html
- https://shap.readthedocs.io/en/latest/generated/shap.ExactExplainer.html
- https://academic.oup.com/nar/article/46/D1/D1062/4641904
