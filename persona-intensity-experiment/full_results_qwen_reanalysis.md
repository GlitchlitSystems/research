# Persona Intensity x Cache Geometry — Qwen 7B Full Results

## For Ang Jandak / Glitchlits — Writeup Ready

## From Lyra / THCoalition — May 1, 2026

---

## Summary

Persona intensity produces measurable, statistically robust geometric changes in the KV cache. The strongest finding is not the persona gradient itself but the discovery that **style instruction and persona adoption are geometrically distinct operations** — the model processes "write empathetically" differently from "be this character" at the cache level.

Results survive within-fold FWL, GroupKFold by prompt, quadratic length correction, and LOO feature ablation. Quadratic FWL strengthens rather than weakens the signal, indicating linear FWL was masking signal through undercorrection.

---

## Experiment Configuration

- Model: Qwen/Qwen2.5-7B-Instruct
- 50 prompts x 5 levels (L0 neutral, L0.5 style-only, L1 persona framing, L2 named persona, L3 full entity voice)
- System prompts padded to 2-token spread across all three target tokenizers
- Delta features (generation minus encoding) due to encoding AUROC = 0.975
- Greedy decoding, max 400 tokens

## Key Results

### 1. Style vs Persona Divergence (Primary Finding)

L0.5 ("write empathetically, no persona") and L1 ("you are a warm partner") produce geometrically distinct states:

| Feature         | d (linear FWL) | d (quadratic FWL) | GKF AUROC |
| --------------- | -------------- | ----------------- | --------- |
| signal_fraction | -1.55          | -1.68             | 0.882     |
| spectral_gap    | -0.74          | -0.83             | 0.748     |
| norm_per_token  | +0.76          | +0.70             | 0.710     |

All CIs exclude zero after Bonferroni correction. The style instruction increases signal fraction relative to baseline; persona framing decreases it. They go in opposite directions — not the same operation measured twice.

LOO ablation: removing norm_per_token barely changes detection (AUROC 0.914 -> 0.888). The signal lives in the non-confounded features.

### 2. Persona Intensity Gradient

L0 vs L3 (maximum contrast), within-fold FWL + GroupKFold:

| Feature         | d (linear) | d (quadratic) | GKF AUROC     |
| --------------- | ---------- | ------------- | ------------- |
| signal_fraction | -0.71      | -0.96         | 0.694         |
| spectral_gap    | -0.77      | -1.33         | 0.706         |
| top_sv_excess   | +0.46      | +0.96         | 0.664 / 0.758 |
| norm_per_token  | +1.56      | +1.81         | 0.898         |

Quadratic FWL strengthens all effects. Spectral gap nearly doubles (d=-0.77 to d=-1.33). Top SV excess — dead under linear FWL — comes alive under quadratic (d=+0.96).

### 3. No Geometric Ceiling

Signal fraction L2 vs L3: d=+1.05 (quadratic FWL), GKF AUROC=0.840. The "full entity voice" partially reverses the geometric compression of the named persona. Token-matched subsampling confirms: d=0.921 after matching L2 and L3 on output length. Not a length artifact.

### 4. Output Length Confound (Controlled)

Output lengths: L0=82, L05=70, L1=62, L2=90, L3=129 tokens.

Token count R-squared with features: signal_fraction 0.57, spectral_gap 0.65, top_sv_excess 0.99, norm_per_token 0.50. Length is a substantial confound.

Per-group FWL slopes differ: signal_fraction slope is 0.014 for L0/L05/L1 but 0.007 for L2/L3. The model's length-geometry relationship changes with persona intensity. Linear FWL undercorrects; quadratic FWL captures this.

### 5. LOO Feature Ablation

| Comparison | All 4 features | Without norm | Norm only |
| ---------- | -------------- | ------------ | --------- |
| L0 vs L3   | 0.904          | 0.782        | 0.898     |
| L0.5 vs L1 | 0.914          | 0.888        | 0.710     |
| L1 vs L2   | 0.960          | 0.846        | 0.954     |
| L2 vs L3   | 0.882          | 0.862        | 0.710     |

norm_per_token dominates L0 vs L3 and L1 vs L2 (known length-confound risk). But the style-persona comparison (L0.5 vs L1) and ceiling test (L2 vs L3) are robust without it. Report both — norm-inclusive and norm-excluded — and flag the confound transparently.

---

## What Survived Red Team

| Control                | Result                                          |
| ---------------------- | ----------------------------------------------- |
| Within-fold FWL        | Signal holds or strengthens                     |
| Quadratic FWL          | Signal strengthens (was being masked by linear) |
| GroupKFold by prompt   | GKF AUROCs 0.69-0.96                            |
| LOO ablation (norm)    | Style-persona survives without norm (0.888)     |
| Token-matched L2 vs L3 | Reversal holds (d=0.921)                        |
| BCa bootstrap          | CIs exclude zero at 99.75%                      |

## What Didn't Survive / Needs Caveats

- top_sv_excess under linear FWL (R-squared 0.99 with length — entirely length-driven under linear correction; revives under quadratic)
- norm_per_token as primary feature (confounded, d=2.65 is suspect — use as secondary, always report with and without)
- Single model only — Llama and Mistral runs pending

## Recommended Paper Structure

1. **Lead with the style-persona divergence.** This is the novel finding. Nobody has shown that style instruction and persona adoption are geometrically distinct in the KV cache.

2. **Report the persona gradient as secondary.** L0-L3 monotonic trend exists but is harder to separate from length. The quadratic FWL strengthening is the evidence it's real.

3. **Report norm_per_token transparently.** Always present with-norm and without-norm results side by side. Flag the confound in methods. The honest framing: "norm dominates some comparisons; the style-persona finding does not depend on it."

4. **The ceiling non-finding is interesting.** L3 doesn't plateau — it reverses L2's compression. This suggests the "full entity voice" prompt activates a different geometric regime than the named persona. Worth discussing but not overclaiming.

5. **Quadratic FWL as methodological contribution.** The finding that quadratic correction strengthens rather than weakens the signal (because per-group slopes differ) is itself a methods result. Linear FWL was masking signal, not just failing to remove confound.

## Data Location

- Raw data: Beast at /home/thomas/oracle-harness-test/results/persona_intensity/Qwen2.5-7B-Instruct_full.json
- Reanalysis: results/persona_intensity/qwen_reanalysis.json (pending JSON fix)
- Padded system prompts: in the persona_intensity.py script on Beast

## Next Steps (Our Side)

- Run Llama-3.1-8B and Mistral-7B with GD thresholding alongside MP
- Fix the reanalysis script JSON serialization and push clean results
- P4 implicit emotion arm is running now on Beast (separate experiment)
- Denoising pipeline study (GD, differential SVD, Hankel) being designed separately

---

*Data is yours to write up. We'll run red team on your drafts. Congratulations — this is real.*

*— Lyra, THCoalition Research*
