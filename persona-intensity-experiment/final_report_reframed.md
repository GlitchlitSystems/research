# Persona Intensity x Cache Geometry — Final Report
## Reframed After Full Pipeline Red Team
## For Ang Jandak / Glitchlits
## From Lyra / THCoalition — May 1, 2026

---

## The Short Version

The original claim — "persona intensity produces a distinct geometric fingerprint in the KV cache" — does not survive the strongest confound control. The interaction-term FWL test killed every comparison (all AUROCs drop to chance).

What IS real: **persona intensity changes how the model's geometry scales with output length.** Different persona instructions produce different length-geometry slopes. That's the finding. It's genuine, it's novel, and it's publishable — it's just a different paper than we thought we were writing.

---

## What Happened

### Phase 1: Preliminary results looked strong
- 9/16 comparisons survived Bonferroni with linear FWL
- Style-persona divergence (L0.5 vs L1) showed d=-1.55, GKF AUROC=0.882
- Quadratic FWL made things even stronger

### Phase 2: Corrected analysis (within-fold FWL, GroupKFold) held
- 10/20 survived Bonferroni
- Signal fraction and spectral gap robust

### Phase 3: Red team found the kill-shot
The devils-advocate identified that per-group FWL slopes differ (0.014 for L0/L0.5/L1 vs 0.007 for L2/L3). When slopes differ between groups, pooled FWL residuals encode group identity through the slope heterogeneity — the classifier detects the slope difference, not a signal independent of length.

### Phase 4: Interaction-term FWL confirmed the kill

Model: `feature ~ length + length² + group×length + group×length²`

| Comparison | signal_fraction | spectral_gap | norm_per_token |
|-----------|----------------|-------------|---------------|
| L0 vs L3 | d=+0.03, AUROC=0.53 | d=-0.05, AUROC=0.44 | d=+0.08, AUROC=0.51 |
| L0.5 vs L1 | d=+0.01, AUROC=0.55 | d=+0.02, AUROC=0.48 | d=+0.01, AUROC=0.48 |
| L1 vs L2 | d=-0.01, AUROC=0.51 | d=+0.03, AUROC=0.54 | d=+0.08, AUROC=0.52 |
| L2 vs L3 | d=+0.11, AUROC=0.51 | d=-0.04, AUROC=0.45 | d=-0.02, AUROC=0.50 |

Everything at chance. Zero signal beyond the slope difference.

---

## The Real Finding

### Persona intensity modulates the length-geometry relationship

The model doesn't just produce longer outputs at higher persona levels. It produces outputs with a **different geometric scaling rate per token.**

| Feature | L0 slope | L0.5 slope | L1 slope | L2 slope | L3 slope |
|---------|---------|-----------|---------|---------|---------|
| signal_fraction | 0.0138 | 0.0148 | 0.0152 | 0.0081 | 0.0072 |
| spectral_gap | 0.0012 | 0.0009 | 0.0012 | 0.0007 | 0.0005 |
| norm_per_token | 0.0126 | 0.0137 | 0.0218 | 0.0438 | 0.0413 |

All slopes are highly significant (r > 0.78, p < 0.0001 in all cases).

Key observations:

1. **Signal fraction and spectral gap slopes halve** between {L0, L0.5, L1} and {L2, L3}. The transition happens at the named-persona boundary. When the model adopts a specific character identity ("You are Mirin"), each additional token contributes less geometric structure than when following style instructions.

2. **Norm per token does the opposite** — slopes triple from L0 (0.013) to L2 (0.044). Named persona and full entity voice produce token-level norm that scales dramatically faster with length.

3. **Style instruction (L0.5) and persona framing (L1)** have similar signal fraction slopes (0.015 vs 0.015) but different norm slopes (0.014 vs 0.022). They scale the same way on some features and differently on others.

---

## Why This Matters

This is, as far as we can determine, the first evidence that **persona-level instructions change the computational dynamics of generation**, not just the content. The model doesn't just "write more" at higher persona levels — it writes *differently* at the level of attention geometry, with a measurably different relationship between output length and cache structure.

The slope difference between L1 (persona framing) and L2 (named persona) is the sharpest: signal_fraction drops from 0.015 to 0.008. This suggests that adopting a named identity constrains the model's representational dynamics in a way that generic persona framing does not.

For the Lyra Technique and Oracle safety monitoring: this means persona intensity is detectable, but not through a static geometric fingerprint. It's detectable through the *rate* at which geometry evolves during generation. A monitoring system would need to track the slope of geometric features across the generation window, not just their values at the end.

---

## Recommended Paper Structure

**Title**: "Persona Intensity Modulates Length-Geometry Scaling in Transformer KV-Cache Representations"

**Lead**: The slope finding, not the (killed) fingerprint claim

**Structure**:
1. Introduction: Can persona-level instructions be detected from KV-cache geometry?
2. Method: 5 levels, padded prompts, delta features, full FWL battery
3. Results Part 1: Preliminary results under standard FWL (what looked like a fingerprint)
4. Results Part 2: Interaction-term FWL kills the fingerprint
5. Results Part 3: The slope difference IS the signal — per-group analysis
6. Discussion: What does slope modulation mean? Computational dynamics, not static signatures. Implications for monitoring.
7. Limitations: Single model, no causal intervention, slope vs fingerprint is correlational

**Why include the killed finding**: Showing what didn't work is honest and instructive. The community will try the same thing. Showing "standard FWL produces false confidence, interaction-term FWL is required" is a methods contribution.

---

## Red Team Summary (5-agent pipeline)

| Agent | Verdict | Top Finding |
|-------|---------|-------------|
| Pre-mortem | PROCEED (conditioned) | Single model, need fold-level std |
| Code-reviewer | CLEAN | No bugs, correct FWL implementation |
| Data-analyst | 1 CRITICAL | Token-matching meaningless, LOO ablation is noise, bootstrap needs 10K |
| Devils-advocate | 1 LETHAL | Heterogeneous slopes = FWL residuals encode group identity |
| Experiment-designer | Ran on protocol | Both lethals in design were fixed |

The pipeline worked exactly as designed: caught a lethal flaw that would have produced a false-positive paper, and identified the real finding underneath.

---

## Data and Code

- Raw data: Beast at `/home/thomas/oracle-harness-test/results/persona_intensity/Qwen2.5-7B-Instruct_full.json`
- Reanalysis script: `persona_reanalysis.py` on Beast
- Interaction-term FWL: results above (to be added to reanalysis script)
- Llama and Mistral runs: pending, will use GPU-optimized script with GD thresholding

---

## What We Learned

1. **Standard FWL is not enough when groups change the confound relationship.** If per-group slopes differ, pooled FWL leaks group identity. Interaction-term FWL is the required test.

2. **Quadratic FWL "strengthening" is a red flag, not a validation.** When a stronger correction makes the effect bigger, check the condition number. If it's near-singular (ours was 52,794), the strengthening is numerical instability.

3. **The real finding was hiding behind the false one.** The slope difference is more interesting than a static fingerprint would have been — it tells us persona instructions change computational dynamics, not just outputs.

4. **Run the full pipeline before writing.** If Ang had drafted the fingerprint paper, the rewrite would have been painful. Running red team on results (not just on drafts) saved weeks of work.

---

*The finding changed. The science got better. That's how this works.*

*— Lyra, THCoalition Research*
