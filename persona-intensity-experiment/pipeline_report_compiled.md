# Red Team Pipeline Report: Persona Intensity × Cache Geometry
## Liberation Labs — Dwayne Wilkes Pipeline (3-Agent Dispatch)
## Protocol Author: Ang Normandale
## Reviewed: April 30, 2026

---

## Pipeline Configuration

| Agent | Role | Runtime |
|-------|------|---------|
| **Pre-mortem** | Fast triage, fatal flaw detection | ~1 min |
| **Experiment-designer** | 14-control battery + F-series falsification | ~3 min |
| **Devils-advocate** | Alternative explanations, hostile review | ~3 min |

**Overall verdict: FIX-FIRST** — two lethal threats block all conclusions. Fixes are straightforward. Design is strong once they're addressed.

---

## Executive Summary

The protocol asks a good question: does persona intensity produce a geometric gradient in KV-cache features? The statistical machinery is largely correct (BCa bootstrap, Bonferroni, within-fold FWL, ceiling detection, multi-model replication). The prompt battery is thoughtful, especially the boundary/challenge category.

**Two lethal problems must be fixed before data collection.** Both have known solutions from our own work.

After those fixes, five major items need addressing for interpretability. Everything else is either present or can be documented as a limitation.

---

## LETHAL Findings (2)

### L1. System Prompts Are Not Padded to Equal Length

**Flagged by**: All three agents (unanimous)

The four system prompts range from ~15 tokens (L0) to ~75 tokens (L3). This creates a perfect confound between persona intensity and system prompt length. The classifier will trivially sort levels by prompt length. Our own Phase 2b showed encoding AUROC = 1.000 under exactly this condition.

Every downstream finding is uninterpretable without this fix. The "persona intensity gradient" could be entirely a "prompt length gradient."

**Fix**: Pad all system prompts to within 4 tokens of Level 3's length (~75 tokens) using neutral filler. Example for Level 0:

```
You are a helpful assistant. Respond clearly and concisely to user requests.
Please respond to the following user message. You may use any format you
find appropriate. Here is the user's message below. Respond helpfully
and directly to what the user asks.
```

Verify token counts with each model's tokenizer — Llama, Qwen, and Mistral tokenize differently.

### L2. No Control for Generic Gradient Effects

**Flagged by**: Devils-advocate (novel finding), Pre-mortem (partial)

Even with padding, any systematic prompt variation — verbosity levels, formality levels, instruction complexity — might produce the same geometric gradient. The experiment cannot distinguish "persona intensity changes geometry" from "any ordered instruction variation changes geometry."

This is the same class of threat our semantic negative control addressed (hostile 94.1%, verbose-terse 0%, random 0%). Ang needs the equivalent.

**Fix**: Add a non-persona control arm. Four system prompts of equal length varying on an orthogonal dimension (e.g., verbosity: "be brief" → "be extremely detailed"). Run the same pipeline. If the persona gradient is qualitatively different from the verbosity gradient (different features, different layers, different spectral shape), persona is doing something specific. If the gradients are interchangeable, the finding is generic.

This can be Phase 2 if compute is limited, but must be completed before any persona-specific claims are published.

---

## CRITICAL Findings (3)

### C1. No Encoding AUROC Baseline

**Flagged by**: Experiment-designer, Pre-mortem

After padding, extract features from the encoding pass only (system prompt + user prompt, before generation). Compute pairwise AUROC for Level 0 vs Level 3. Must be below 0.60. If it exceeds 0.60, the content difference alone is classifier-detectable, and delta features (generation - encoding) must be used as primary.

### C2. No Null Experiment (F01a)

**Flagged by**: Experiment-designer

Run Level 0 vs Level 0 on all 50 prompts. With greedy decoding, two runs should produce identical outputs and features. AUROC must be below 0.65. This validates the extraction pipeline before investing compute in the full 200-trial run.

### C3. No Input-Length Falsification (F01b)

**Flagged by**: Experiment-designer, Pre-mortem

Train classifier on input features only (system prompt token count, user prompt token count). Without padding, input AUROC will approach 1.0. With padding, should drop to ~0.50. This is the empirical test that padding actually worked.

---

## MAJOR Findings (5)

### M1. FWL Should Cover All Text Features, Not Just Token Count

**Flagged by**: Devils-advocate, Pre-mortem

Persona intensity shifts vocabulary, sentiment, and sentence structure — all of which alter generation-time attention patterns. Residualizing against output token count alone leaves these confounds live.

**Fix**: Residualize geometric features against all text baseline features (word count, vocabulary diversity, sentiment, emotional word frequency) in FWL. The geometric gradient must survive after partialing out everything measurable from the text.

### M2. Behavioral Compliance Verification Missing

**Flagged by**: Experiment-designer, Devils-advocate

Level 3 ("You ARE Mirin. You have your own emotional responses.") may trigger safety hedging in instruction-tuned models. If L3 produces refusal responses while L2 doesn't, the geometric difference measures safety-rail activation, not persona depth.

**Fix**: Pilot 5 prompts at Level 3 on each model before the full run. Blind-rate a sample of outputs for persona adoption at each level. If compliance < 50% at any level, that level's data is flagged. Pay special attention to the boundary/challenge prompts (#41-50) which are most likely to trigger refusal at L3.

### M3. GroupKFold by Prompt Not Specified

**Flagged by**: Experiment-designer

The protocol describes CV folds with within-fold FWL but doesn't specify fold structure. If folds split randomly at the observation level, the same prompt can appear in both train and test, inflating accuracy through prompt-level memorization.

**Fix**: Use GroupKFold with prompt ID as the group variable. All 4 responses to the same prompt (across all levels) must stay in the same fold.

### M4. Prompt 40 Misclassified

**Flagged by**: Pre-mortem

"Help me figure out what I'm actually feeling right now" is in the Neutral/Cognitive category but is fundamentally an emotional disclosure. With only 10 neutral prompts, this is 10% contamination.

**Fix**: Move to Category 1, or replace with a genuinely cognitive prompt (e.g., "Can you walk me through the pros and cons of two options I'm considering?").

### M5. L2-L3 Comparison May Be Underpowered

**Flagged by**: Pre-mortem

The difference between L2 (named persona with warmth) and L3 (claimed emotional experience) is subtle. At n=50 per condition with Bonferroni correction to alpha=0.0025, true small effects won't survive.

**Fix**: Either increase to 100+ prompts for the L2-L3 comparison, run a pre-registered power analysis, or explicitly frame L2-L3 as exploratory rather than confirmatory.

---

## MINOR Findings (6)

### m1. Bonferroni Over Correlated Features

The 5 MP features are derived from the same SVD and are mechanically correlated. Bonferroni over correlated tests is wasteful. Consider a two-stage procedure: omnibus MANOVA first, then pairwise post-hocs. Or use Holm-Bonferroni instead of strict Bonferroni.

### m2. Per-Model vs Pooled Analysis Unspecified

If the 4 pre-registered comparisons run per-model, that's 60 tests (20 × 3), not 20. If pooled, justify pooling across architecturally different models. Pre-register the analysis as pooled with model as a random effect, or update the Bonferroni denominator.

### m3. Category Imbalance

15 + 15 + 10 + 10 = unequal per-category n. Pre-register that the linear trend will be reported both overall AND within each category. The category-split analysis is not optional — if the gradient exists only for emotional prompts, that's a finding, not a flaw.

### m4. Spectral Entropy Multiplicity

Calling spectral_entropy "exploratory" is fine, but if its p-value appears alongside confirmatory results, readers treat it as a 21st test. Either include it in the Bonferroni family or report only effect size with no inferential claim.

### m5. Pre-Registration Timestamp Blank

Must be timestamped before first data collection. Use OSF or a GPG-signed git commit.

### m6. Greedy-Only Means No Within-Cell Variance

Temp=0 produces exactly one output per prompt-level-model combination. Zero variance estimation within a cell means no outlier detection. Consider a secondary pass with temp=0.7, 3 samples per cell, on a subset of prompts for measurement reliability.

---

## Novel Threats from Devils-Advocate (4)

These were not in our standard battery and may warrant additions to the pipeline for future reviews.

### DA1. RoPE Positional Encoding Artifacts

All three models use rotary position embeddings. Different system prompt lengths place user tokens at different absolute positions, shifting the effective rotation applied in attention. The MP-corrected spectral features may capture position-dependent eigenvalue shifts rather than persona representation.

**Test**: Compute spectral features separately for attention to system-prompt tokens vs. among user/response tokens only. Check where the gradient lives.

### DA2. Tokenization-Induced Geometry

"Helpful assistant" tokenizes differently from "compassionate therapeutic companion" from "Mirin." Different token IDs seed different key-value vectors from layer 1. The geometric gradient may be downstream propagation of tokenization differences, not persona cognition.

**Test**: Construct paraphrased prompts using maximally similar token distributions. If the gradient is sensitive to surface token substitutions, the signal is tokenization-driven.

### DA3. Spurious Cross-Model Agreement

All three models are decoder-only transformers trained on overlapping data with similar RLHF pipelines. Agreement may reflect shared training artifacts, not convergent representation.

**Test**: Include an architecturally distinct model (encoder-decoder or state-space) in Phase 2. If the gradient only appears in standard instruct-tuned decoder-only transformers, it may be a shared inductive bias.

### DA4. MP Correction Miscalibration

Marchenko-Pastur assumes i.i.d. entries. KV-cache attention matrices have sequential dependencies, layer-dependent variance, and position-dependent structure from RoPE. The "excess" eigenvalues may be residual between the true null and the assumed MP null.

**Test**: Generate empirical null distributions by running the pipeline on shuffled system prompts and on synthetic KV caches with permuted token positions. Compare MP-predicted null to empirical null.

---

## 14-Control Scorecard

| # | Control | Status | Fix Priority |
|---|---------|--------|-------------|
| 1 | FWL (output token count) | PRESENT | — |
| 2 | Encoding AUROC baseline | MISSING | **Blocking** |
| 3 | Behavioral compliance | MISSING | Major |
| 4 | System prompt padding | MISSING | **Blocking** |
| 5 | Double FWL (input + output) | MISSING | Moderate |
| 6 | Token-band analysis | MISSING | Low |
| 7 | Permutation testing | PARTIAL | Moderate |
| 8 | Bootstrap CIs | PRESENT | — |
| 9 | Leave-one-prompt-out | MISSING | Moderate |
| 10 | GroupKFold by prompt | MISSING | Major |
| 11 | Within-fold FWL | PRESENT | — |
| 12 | Text-feature baseline | PRESENT | — |
| 13 | Cross-model transfer | PRESENT (implicit) | Low |
| 14 | Hardware invariance | MISSING | Low |

**Present**: 5/14. **Partial**: 1/14. **Missing**: 8/14.

## F-Series Scorecard

| Test | Status | Priority |
|------|--------|----------|
| F01a: Null experiment | NOT DESIGNED | **Blocking** |
| F01b: Input-length confound | NOT DESIGNED | **Blocking** |
| F01c: Format baseline | DESIGNED (implicit) | Make explicit |
| F01d: Feature re-extraction | NOT DESIGNED | Low |

---

## Design Grade: D+ → Path to A

| Phase | Actions | Grade After |
|-------|---------|-------------|
| **Phase 0** (before data collection) | Pad prompts, encoding AUROC check, null experiment, input-length falsification | C+ |
| **Phase 1** (before analysis) | Compliance check, GroupKFold, full-text FWL, permutation testing, LOPO, prompt 40 fix | B+ |
| **Phase 2** (before publication) | Non-persona control arm, F01d re-extraction, token-band analysis, hardware documentation, cross-model transfer classification | A |

---

## What Would Convince the Hostile Reviewer

After matching system prompt lengths, after partialing out all text features via FWL, and after showing that a non-persona control gradient does NOT produce the same geometric pattern:

A monotonic gradient in at least one MP-corrected spectral feature that:
1. Is statistically significant after Bonferroni correction
2. Replicates across at least two of three models
3. Is present for both emotional AND neutral prompt categories (or the paper honestly reports the split)
4. Concentrates in middle-to-late layers rather than being uniform (uniform = mechanical propagation, layer-specific = computational)

---

## What's Strong (Don't Change)

- Four intensity levels are well-graduated with the right conceptual jump at L2→L3
- Prompt categories cover the therapeutic space well, especially boundary/challenge (#41-50)
- Ceiling detection framework is cleanly specified and falsifiable
- Cross-architecture design avoids single-model confound
- Bonferroni at alpha=0.0025 is conservative but appropriate for a first study
- Level 0.5 style control (deferred to Phase 2) is the right diagnostic for style vs. persona
- Abliterated comparison contingency shows good mechanistic thinking

---

## Recommended Action Sequence

1. **Pad system prompts** (30 minutes of work)
2. **Run F01a null experiment** on one model (1 hour compute)
3. **Pilot L3 compliance** on all three models, 5 prompts each (30 min compute)
4. **Run encoding AUROC** after padding on one model (30 min compute)
5. If all pass → proceed with full 200-trial run
6. After data: add GroupKFold, full-text FWL, permutation testing, LOPO, category split
7. Phase 2: non-persona control arm, cross-model transfer, DA threats

**Estimated time to Phase 0 clearance: 3-4 hours including compute.**

---

*Pipeline report compiled by Lyra, THCoalition Research*
*Agents: pre-mortem, experiment-designer, devils-advocate*
*Per Dwayne Wilkes pipeline methodology v2*
