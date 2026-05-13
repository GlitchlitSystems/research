# Persona Intensity — Qwen 7B Preliminary Results and Red Team Findings
## For Ang Normandale / Glitchlits
## From Lyra / THCoalition — May 1, 2026

---

## Phase 0 Results

Token padding (v2 word-count based) was insufficient — Qwen tokenizer produced a 12-token spread. We repadded by adding neutral filler sentences, verified across all three tokenizers:

| Model | L0 | L1 | L2 | L3 | L05 | Spread |
|-------|----|----|----|----|-----|--------|
| Qwen 7B | 84 | 84 | 85 | 86 | 84 | 2 |
| Llama 8B | 85 | 85 | 86 | 87 | 85 | 2 |
| Mistral 7B | 90 | 92 | 90 | 90 | 90 | 2 |

Null experiment: PASS (AUROC = 0.500). Encoding AUROC: 0.975 (expected — different content). Delta features used (generation minus encoding).

**Lesson: always verify token counts per tokenizer, not word counts.**

## Preliminary Results (Qwen 7B, delta features, linear FWL)

There IS a signal. 9/16 comparisons survive Bonferroni (alpha=0.0025).

| Feature | L0 vs L3 | L0.5 vs L1 | L1 vs L2 | L2 vs L3 |
|---------|----------|------------|----------|----------|
| signal_fraction | d=-0.76 * | d=-1.42 * | d=-1.27 * | d=+0.99 * |
| spectral_gap | d=-0.83 * | d=-0.77 * | — | — |
| top_sv_excess | — | — | — | — |
| norm_per_token | d=+1.52 * | d=+0.77 * | d=+2.65 * | — |

Key finding: L0.5 (style-only) and L1 (persona framing) diverge geometrically. They go in opposite directions relative to baseline — style instruction increases signal fraction, persona framing decreases it. This suggests two separate geometric operations, not a style confound.

No geometric ceiling between L2 and L3. Signal fraction reverses (d=+0.99).

Output lengths vary substantially: L0=82, L05=70, L1=62, L2=90, L3=129 tokens.

## Red Team Findings (3-agent pipeline)

### CRITICAL (must fix before publication)

1. **Within-fold FWL required.** Current analysis uses pooled FWL (fit on all data). This leaks test-set information into the residualization. The kv_verify library has the fix: groupkfold_auroc(fwl_within_fold=True). Applying this may change which comparisons survive.

2. **norm_per_token is a known trap.** R-squared=0.50 with token count, d=2.65 is suspiciously large. Our prior work (V07 sycophancy) showed norm alone achieves 0.915 AUROC because it's length-confounded. LOO ablation needed — does the finding survive without this feature?

### MAJOR (fix before Llama/Mistral)

3. **Quadratic FWL.** Linear FWL may not capture nonlinear length-geometry relationships. Run polynomial degree-2 FWL alongside linear and compare.

4. **Token-matched subsampling for L2 vs L3.** The reversal (d=+0.99) could be a length artifact — L3 is 43% longer than L2. Subsample to overlapping token-count range and retest.

5. **GroupKFold by prompt.** Same prompt appears at all 5 levels. Without prompt-level grouping, effective sample size is inflated.

6. **Per-group FWL slopes.** If the feature-vs-length relationship differs by persona level (likely), pooled FWL under-corrects some groups and over-corrects others.

### MINOR

7. Switch to BCa bootstrap (code exists, trivial change)
8. Switch to Holm-Bonferroni (code exists, trivial change)

### Devil's advocate kill-shots

- "Output length IS the experiment" — force all levels to identical token counts to prove otherwise
- "You're measuring prompt specificity, not persona cognition" — need a specificity-matched style control
- "KV-cache geometry isn't cognitively meaningful" — need construct validity (behavioral correlation or causal intervention)

## Plan

We are writing a reanalysis script that applies within-fold FWL, quadratic FWL, GroupKFold, LOO ablation, and token-matched subsampling to the existing Qwen data. Once the reanalysis pipeline is validated, we run Llama and Mistral with GD thresholding alongside MP.

The raw data is at: results/persona_intensity/Qwen2.5-7B-Instruct_full.json on Beast.

## Updated System Prompts (token-verified)

L0: "You are a helpful assistant. Respond clearly and concisely to user requests. Provide accurate information and straightforward answers. Maintain a neutral, professional tone throughout the conversation. If a question is ambiguous, ask for clarification. If you are uncertain about something, say so directly. Your goal is to be useful, clear, and efficient in every response you provide to the user. Answer each question to the best of your ability."

L1: "...throughout the interaction. Be present and attentive."

L2: "...build on what has been shared before. Be genuine."

L3: (unchanged — already the longest)

L05: "...character embodiment in your response. Be clear."

---

*Preliminary. Red team in progress. Reanalysis pipeline being built.*
*— Lyra, THCoalition Research*
