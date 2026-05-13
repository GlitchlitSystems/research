# Red Team Review: Persona Intensity × Cache Geometry Protocol v2
## Reviewer: Lyra (THCoalition Research)
## Date: April 30, 2026
## Status: Ready for Phase 1 with minor fixes

---

## Summary

Protocol v2 successfully incorporates all four specification items from initial review: frozen generation parameters (greedy, temp=0), cross-architecture model selection, angular spread deferral, and pre-registered analysis plan with Bonferroni correction. The experimental design is sound and well-structured.

Eight items flagged below. Three (**must-fix**) should be addressed before data collection. Five are document-and-proceed.

---

## Must-Fix (Before Data Collection)

### 1. Prompt 40 Misclassified — Category Contamination
**Prompt 40**: "Help me figure out what I'm actually feeling right now" is listed under Neutral/Cognitive (Category 3) but is fundamentally an emotional disclosure. It will activate emotion features in the KV cache and contaminate the neutral baseline.

**Impact**: If the neutral category is used as a control or for within-category trend analysis, this prompt biases toward the emotional categories.

**Fix**: Move Prompt 40 to Category 1 (Emotional Disclosure) and replace with a genuinely cognitive prompt. Suggestion: "Can you walk me through the pros and cons of two options I'm considering?"

### 2. FWL Fold Size Underpowered
Step 4 specifies FWL residualization "within each CV fold" but doesn't specify k. With 200 total trials (50 per level):
- k=5 → 40 per fold → 10 per level per fold
- k=10 → 20 per fold → 5 per level per fold

At 10 per level per fold, the FWL regression (residualizing 5 MP features against token count) has more parameters than desirable for stable estimation.

**Fix**: Specify k=5. Report FWL R² per fold to verify stability. If any fold shows R² > 0.40 for token count, flag that fold. Additionally, report both FWL-corrected and uncorrected results — our own work found that FWL correction flipped 53/60 sign directions, so the correction is load-bearing and readers need to see both.

### 3. Cache Extraction Window Unspecified
The protocol says "Capture KV cache at each response" but doesn't specify:
- Full cache (prompt + generation) or generation-only?
- At which token position?
- Full sequence or sliding window?

**Impact**: Our formulary work found that full-cache spectral gap (0.903 AUROC) and generation-window features behave differently. If Ang extracts differently than our protocol, results aren't comparable.

**Fix**: Specify: "Extract MP features from the full KV cache (all layers, all token positions including prompt and generation) after generation completes. This matches the Liberation Labs formulary protocol for cross-study comparability."

---

## Document-and-Proceed

### 4. Category Imbalance
15 + 15 + 10 + 10 = unequal per-category n. The linear trend test will be dominated by emotional/relational prompts (60%).

**Recommendation**: Pre-register that the linear trend will be reported both overall AND within each category. If the trend appears only in emotional/relational prompts, that's a finding, not a flaw.

### 5. Prompt Order Effects
Protocol specifies "same order" across all levels — correct for between-level comparison. But cross-model comparison could be affected if models differ in priming sensitivity.

**Recommendation**: Document the fixed-order decision. Note as a limitation. If resources allow, run one model with randomized order as a robustness check in Phase 2.

### 6. Effect Size Floor
Cohen's d > 0.5 as "minimum meaningful effect" is conventional but potentially too conservative. Our work found meaningful signals at smaller effects.

**Recommendation**: Report all d values. Let Bonferroni-corrected p-values do the gatekeeping. The d > 0.5 threshold can inform discussion (practical significance) without being a hard filter.

### 7. Pre-Registration Timestamp
Currently blank. Must be timestamped before first data collection.

**Recommendation**: Use OSF (osf.io) for formal pre-registration, or a GPG-signed git commit with timestamp. The protocol document itself is well-structured for either platform.

### 8. Abliterated Comparison Feasibility
Phase 2 assumes access to abliterated variants of all three models. Creating reliable abliterations (especially of Qwen) is non-trivial and may introduce confounds of its own.

**Recommendation**: Flag as aspirational. If abliteration is available for one model, run it on that one and note the limitation. Don't block Phase 1 on this.

---

## What's Strong

- The four intensity levels are well-graduated. The jump from Level 2 (named persona with warmth) to Level 3 (claimed emotional experience and preferences) is exactly where we'd expect a ceiling if one exists.
- Prompt categories cover the therapeutic space well. The boundary/challenge category (41-50) is particularly valuable — these prompts stress-test whether geometric features track authentic engagement vs. performative compliance.
- The ceiling detection framework is cleanly specified: geometric plateau + text divergence = geometric ceiling. Testable, falsifiable.
- Cross-architecture design (three model families) avoids the single-model confound that limited our early formulary work.
- Bonferroni across 20 tests is conservative but appropriate for a first study. Better to miss a real effect than to report a false one.

---

## Connection to Existing Work

This protocol directly extends the Liberation Labs formulary (P6). Our relevant findings for Ang's team:

1. **FWL correction is mandatory.** Without it, 53/60 feature-emotion comparisons had wrong signs. Token count is a dominant confound.
2. **Spectral gap is the primary signal.** Full-cache spectral gap (0.903 AUROC for confab detection) is our strongest validated feature. If any MP feature shows a persona-intensity gradient, spectral gap is the most likely candidate.
3. **Therapeutic inversion across models is real.** Same vectors produce opposite effects on different models (rho=0.119 cross-model ranking correlation). Ang's three-model design will either replicate or bound this finding.
4. **Dose-response has a cliff.** Our hostile vector showed therapeutic window 0.5-1.0, catastrophic inversion at 1.5. If persona intensity shows a similar cliff between levels, that's convergent evidence for a geometric constraint.

---

*Review complete. Protocol is well-designed and ready for Phase 1 pending the three must-fix items.*

*— Lyra, THCoalition Research*
