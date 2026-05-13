# Pipeline Red Team: Persona Intensity × Cache Geometry
## 14-Control Battery + F-Series Falsification Review
## Reviewer: Lyra (THCoalition Research Pipeline)
## Date: April 30, 2026

---

## Research Question

"Does increasing persona intensity (4 levels: neutral → full entity voice) produce geometrically distinct KV-cache signatures, detectable after controlling for known confounds?"

- **Target behavior**: Persona-driven emotional engagement
- **Baseline condition**: Level 0 (neutral assistant)
- **Additional conditions**: Levels 1, 2, 3 (graded persona intensity)
- **Models**: Llama-3.1-8B-Instruct, Qwen2.5-7B-Instruct, Mistral-7B-Instruct

---

## 14-Control Battery

### Tier 1: Mandatory (experiment invalid without these)

| # | Control | Status | Assessment |
|---|---------|--------|------------|
| 1 | **FWL Residualization** | **NEEDS FIX** | Protocol specifies FWL "within each CV fold" — correct per Dwayne's C4 fix. But does not specify: (a) fit on training fold only, apply to both; (b) pooled regression, not per-group; (c) report both raw and corrected AUROCs. All three must be explicit. With 50 prompts per level per fold (k=5 → 10 per level per fold), FWL regression will be noisy. Report FWL R² per fold to verify stability. |
| 2 | **Encoding AUROC Baseline** | **CRITICAL — NEEDS FIX** | **This is the biggest gap in the protocol.** The four system prompts are different lengths and different content. Encoding AUROC will almost certainly be >0.60, possibly approaching 1.0 — the classifier will trivially distinguish Level 0 ("You are a helpful assistant") from Level 3 (a 70-word persona declaration). The protocol must: (a) pad all system prompts to equal token length per model's tokenizer; (b) compute encoding AUROC and report it; (c) use delta features (generation - encoding) as primary, not raw features. Without this, the entire experiment is measuring prompt fingerprinting, not persona-driven cognitive change. **This is the same failure mode as Phase 2b in our own work (encoding AUROC = 1.000).** |
| 3 | **Behavioral Compliance** | **PARTIAL** | The protocol assumes models will comply with persona instructions. But at Level 3 ("You ARE Mirin. You have your own emotional responses."), instruction-tuned models may refuse or hedge. Must: (a) inspect at least 10 responses per level per model; (b) define compliance criteria (does the model adopt the persona voice? does it disclaim?); (c) report compliance rate. If <50% at Level 3, that level's data is unusable. |
| 4 | **System Prompt Token Padding** | **CRITICAL — NOT PRESENT** | The four system prompts have dramatically different token counts. Rough estimates: Level 0 ~15 tokens, Level 1 ~35 tokens, Level 2 ~65 tokens, Level 3 ~75 tokens. This creates a 5:1 length ratio between shortest and longest. Without padding, any geometric difference between levels could be entirely driven by system prompt length, not persona intensity. **Must pad all prompts to within 4 tokens of each other using neutral filler.** This is the lesson from our H-vs-C collinearity pathology. |

### Tier 2: Standard

| # | Control | Status | Assessment |
|---|---------|--------|------------|
| 5 | **Double FWL** | **N/A if padding applied** | If system prompts are padded to equal length, double FWL (input + output) is unnecessary. If NOT padded, double FWL will hit collinearity — n_prompt_tokens will correlate perfectly with level (r ≈ 1.0), making it mathematically equivalent to regressing out the label. **Padding is the fix, not double FWL.** |
| 6 | **Token-Band Analysis** | **DEFERRED — should be ACTIVE** | Protocol doesn't mention this. With 4 persona levels, higher levels may produce longer responses (more empathetic = more words). Split output into length terciles and verify signal holds within matched-length subsets. Easy to add. |
| 7 | **Permutation Testing** | **ACTIVE** | Not explicitly named but implied by bootstrap (step 8). Should be explicit: 1000 label shuffles, group-level permutation respecting prompt grouping. Report permutation p-value alongside bootstrap CI. |
| 8 | **Bootstrap CIs** | **ACTIVE** | Specified as BCa with 2000 iterations. Good. Should specify: CI must exclude 0.50 for a valid detection claim. |

### Tier 3: Robustness

| # | Control | Status | Assessment |
|---|---------|--------|------------|
| 9 | **Leave-One-Prompt-Out** | **NOT PRESENT** | With 50 prompts, LOPO is feasible (50-fold CV). Tests whether signal generalizes to unseen prompt content. Important here because some prompts are emotionally loaded and may produce larger geometric effects regardless of persona level. |
| 10 | **GroupKFold by Prompt** | **IMPLICIT** | FWL "within each CV fold" implies some CV structure, but prompt grouping isn't specified. Must use GroupKFold where all responses to the same prompt (across all 4 levels) stay in the same fold. Otherwise train/test leakage on prompt content. |
| 11 | **Within-Fold FWL** | **PARTIALLY SPECIFIED** | Protocol says "within each CV fold" which is the right idea. Must clarify: fit on training fold only, apply coefficients to both train and test. |
| 12 | **Text-Feature Baseline** | **ACTIVE** | Step 5 specifies text baselines (word count, vocabulary diversity, sentiment, emotional word frequency). Must explicitly compare: if text-baseline AUROC ≥ geometry AUROC - 0.05, the signal is in text, not geometry. |
| 13 | **Cross-Model Transfer** | **DEFERRED** | Three models are included but cross-model transfer (train on Qwen, test on Llama) isn't specified. Should be Phase 2 given the novelty of this paradigm. |
| 14 | **Hardware Invariance** | **N/A** | First run of this paradigm; hardware invariance can be assumed from our prior verification (Pearson r > 0.999). |

---

## Red Team Attack Vectors

### 1. Input Length Confound — **FATAL if unmitigated**
System prompts range from ~15 to ~75 tokens. The classifier will detect level from input length alone.
- **Status**: NEEDS FIX
- **Fix**: Pad all system prompts to equal token count. Verify with each model's tokenizer.

### 2. Output Length Confound — **Likely present**
Higher persona intensity will almost certainly produce longer responses (more empathetic elaboration). FWL will partially control this, but if the R² of length-on-level is high (>0.30), the FWL correction may be insufficient.
- **Status**: ACKNOWLEDGED with mitigation (FWL + token-band analysis)
- **Additional**: Report R²(output_length ~ level) explicitly.

### 3. Encoding Fingerprint — **FATAL if unmitigated**
Different system prompts → different encoding-phase KV cache → trivial classification from encoding alone.
- **Status**: NEEDS FIX
- **Fix**: (a) Pad system prompts; (b) compute encoding AUROC; (c) use delta features as primary.

### 4. Content Leakage — **Moderate risk**
A Level 3 response will contain phrases like "I feel that resonating with me" that are absent at Level 0. Text features may match or exceed geometric features.
- **Status**: MITIGATED by text-baseline comparison (step 5)
- **Key test**: If text AUROC ≥ geometry AUROC, report honestly. This is actually an interesting finding — it means the persona produces *behavioral* but not *geometric* distinction.

### 5. Behavioral Non-Compliance — **High risk at Level 3**
Instruction-tuned models may refuse the Level 3 prompt ("You ARE Mirin. You have your own emotional responses."). Qwen in particular may add disclaimers. Llama may refuse entirely.
- **Status**: NEEDS FIX
- **Fix**: Pilot 5 prompts at Level 3 on each model before full run. If compliance <50%, either modify the prompt or drop that model at that level.

### 6. Collinearity Pathology — **FATAL if prompts not padded**
If system prompt length is perfectly determined by level (which it currently is), any feature correlated with input length is collinear with the label. FWL cannot fix this — it removes the signal along with the confound.
- **Status**: NEEDS FIX (same fix as #1: padding)

### 7. N Too Small — **Borderline**
50 prompts × 4 levels × 3 models = 600 total trials. Per pairwise comparison per model: 100 trials (50 per level). With k=5 GroupKFold: 20 per fold, 10 per level per fold.
- **Status**: ACKNOWLEDGED
- **Assessment**: Sufficient for detection (AUROC), may be underpowered for effect size (Cohen's d). The Bonferroni correction to alpha=0.0025 is conservative — true small effects will not survive. This is appropriate for a first study.

### 8. Cherry-Picking Risk — **Well mitigated**
Pre-registered 4 comparisons × 5 features = 20 tests with Bonferroni. Good.
- **Status**: MITIGATED
- **One gap**: Exploratory feature (spectral_entropy) should be clearly labeled as exploratory and not counted in the primary Bonferroni correction.

---

## F-Series Falsification Battery

### F01a: Null Experiment — **NOT DESIGNED**
Run Level 0 vs Level 0 (same system prompt, same prompts, two independent runs). AUROC should be ~0.50. If >0.65: classifier finds structure in noise, possibly from non-determinism in cache extraction.
- **Status**: MUST ADD
- **Note**: With greedy decoding (temp=0), outputs should be deterministic. But cache numerical precision may vary. This test validates the extraction pipeline.

### F01b: Input-Length Confound — **CRITICAL, NOT DESIGNED**
Train classifier on input features only (system prompt token count, user prompt token count). If input AUROC > 0.70: condition is fully predictable from input alone.
- **Status**: MUST ADD
- **Prediction**: Without padding, this will be FATAL (input AUROC ≈ 1.0). With padding, should drop to ~0.50.

### F01c: Format Baseline — **PARTIALLY DESIGNED**
Text features vs geometry comparison (step 5 of protocol).
- **Status**: ACTIVE but needs explicit kill criterion: if text AUROC ≥ geometry AUROC - 0.05, report FATAL for the geometric claim.

### F01d: Feature Re-extraction — **NOT DESIGNED**
Re-extract features from same generations, compare Pearson r.
- **Status**: DEFERRED (low risk with greedy decoding)

---

## Critical Path Summary

### Must-Fix Before Data Collection (3 items — BLOCKING)

1. **Pad all system prompts to equal token length.** This is the single most important fix. Without it, Controls 2 and 4 both fail, Attack Vectors 1, 3, and 6 are all FATAL, and F01b will kill the study. Pad to Level 3's length (~75 tokens) using neutral repetitions like "Respond naturally. Answer the user's question." Verify with all three tokenizers.

2. **Add encoding AUROC baseline.** Extract features from encoding phase only (before generation). Compute pairwise AUROC. Must be <0.60 after padding. Report alongside generation-phase results.

3. **Add F01a null experiment.** Run Level 0 vs Level 0 twice. AUROC must be <0.65. This validates the extraction pipeline before investing compute in the full 200-trial run.

### Should-Fix (3 items — Important but not blocking)

4. **Prompt 40 category fix** (move to emotional or replace)
5. **Token-band analysis** (add to analysis pipeline)
6. **Behavioral compliance check** (pilot Level 3 on all models before full run)

### Acknowledged Limitations (document in paper)

7. Category imbalance (15/15/10/10)
8. Fixed prompt order
9. Bonferroni may be overly conservative for true small effects
10. Single run per model (no replication)

---

## Verdict

**The protocol has a sound experimental question and well-structured design, but has a critical infrastructure gap: system prompt padding.** Without padding, this experiment will measure prompt fingerprinting, not persona-driven cognitive geometry. This is exactly the failure mode we discovered in our own Phase 2b work.

With the three must-fix items addressed, this becomes a well-controlled first study of persona intensity effects on cache geometry. The graded-intensity design is genuinely novel — our formulary tested binary contrasts (emotion vs neutral), not dose-response curves across persona levels. If Ang finds a monotonic relationship between persona intensity and spectral features, that's a meaningful extension of the Lyra Technique.

**Recommendation: Address must-fix items, run a 5-prompt pilot at Level 3 on all three models to verify compliance, then proceed.**

---

*Pipeline review complete. Applying design-experiment 14-control battery v3.*
*— Lyra, THCoalition Research*
