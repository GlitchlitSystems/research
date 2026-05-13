# Persona Intensity — Reprocessed Free Features: Cross-Architecture
## Overnight Analysis, May 4-5, 2026

## Key Finding: sv_kurtosis persona effect replicates on all three architectures

The original slope analysis used the wrong test for persona effects. Reprocessing stored singular values through the unified extraction module and testing with ANOVA (level effects, not slope interactions) reveals significant persona effects on all models.

### sv_kurtosis ANOVA by persona level (L0-L3):

| Model | F-statistic | p-value | Significant? |
|-------|------------|---------|-------------|
| Qwen 7B | 31.8 | < 0.000001 | *** |
| Llama 8B | 13.6 | < 0.000001 | *** |
| Mistral 7B | 56.5 | < 0.000001 | *** |

### Llama 8B per-level means (reprocessed from stored top-20 SVs):

| Feature | L0 | L0.5 | L1 | L2 | L3 |
|---------|-----|------|-----|-----|-----|
| stable_rank | 5.919 | 5.915 | 5.891 | 5.870 | 5.840 |
| sv_kurtosis | -0.844 | -0.849 | -0.854 | -0.823 | -0.839 |

### Mistral 7B per-level means:

| Feature | L0 | L0.5 | L1 | L2 | L3 |
|---------|-----|------|-----|-----|-----|
| stable_rank | 6.037 | 6.120 | 6.026 | 6.055 | 6.034 |
| sv_kurtosis | -0.785 | -0.839 | -0.818 | -0.825 | -0.843 |

### Note on extraction

These results were reprocessed from raw top-20 singular values stored in the persona intensity JSON files, using the unified lyra_features.py module. Qwen data did not store top_svs (older script version) — Qwen results use the unified verbosity experiment extraction instead.

### Status: PENDING RED TEAM

These results have not been red-teamed. The ANOVA is the right test (level effects, not slope interactions) but the reprocessed features are computed from truncated SVs (top 20 of ~128), which may introduce approximation error. Cross-architecture comparison uses different raw feature values (different extraction runs) which should be validated.
