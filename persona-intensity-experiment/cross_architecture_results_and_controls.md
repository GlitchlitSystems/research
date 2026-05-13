# Persona Intensity — Cross-Architecture Results + Red Team + Path to Publication
## For Ang Jandak / Glitchlits
## From Lyra / THCoalition — May 3, 2026

---

## Cross-Architecture Results (All Delta Features)

All three models now use matched extraction (delta = generation minus encoding).

### The Universal Pattern

Every model shows the same three-step result:
1. Strong effects under linear FWL (d = 0.76 to 2.83)
2. ALL effects killed by interaction-term FWL (all d < 0.12)
3. Per-group slopes genuinely differ between persona levels

### Per-Model Summary

| | Qwen 7B | Llama 8B | Mistral 7B |
|-|---------|----------|------------|
| Active feature | signal_fraction delta | spectral_gap delta | spectral_gap delta |
| Linear FWL L0 vs L3 | d=-0.76 | d=-2.83 | d=+1.45 |
| Style vs Persona | d=-1.42 | d=+1.91 | d=+2.19 |
| Interaction FWL | ALL DEAD | ALL DEAD | ALL DEAD |
| Slopes modulated? | Yes (halves at L2) | Yes (flips sign at L2) | Yes (flips sign) |
| signal_fraction delta | Varies (primary) | FLAT (MP saturated) | FLAT (MP saturated) |
| Output length range | 62-129 tokens | 108-159 tokens | 176-243 tokens |

### What Replicates and What Doesn't

**Replicates:** The interaction-term kill (slopes ARE the signal) and the existence of slope heterogeneity between persona levels.

**Doesn't replicate:** The specific feature. Qwen shows it in signal_fraction, Llama and Mistral in spectral_gap. signal_fraction delta is flat on Llama/Mistral because MP threshold saturates even on delta features.

---

## Red Team Findings (5-agent pipeline)

### Verdict: NEEDS WORK (trending INSUFFICIENT without controls)

Three critical gaps must be addressed before publication:

### Gap 1: Length-Matched Slope Comparison (CRITICAL)
The slope differences could just reflect "longer outputs scale differently in their geometry." Bin outputs by length (e.g., quartiles), test slopes within matched-length bins. If persona slopes still differ within 100-120 token outputs, the finding is real. If they converge within length bins, it's a confound.

### Gap 2: Non-Persona Specificity Control (CRITICAL)
"Slopes differ between persona levels" is potentially true of ANY prompt variation. You need a control arm with non-persona prompt variation that produces similar length distributions. For example, 4 levels of verbosity instruction ("be brief" → "be extremely detailed") padded to the same token counts as the persona prompts. If persona slopes are qualitatively different from verbosity slopes (different features, different magnitudes, different patterns), persona is doing something specific. If they look the same, the finding is generic.

This is the same logic as the semantic negative control that validated the formulary: hostile corrected 94%, verbose-terse corrected 0%. The specificity test is what separates a finding from an artifact.

### Gap 3: Post-Hoc Feature Selection (MAJOR)
Three models, three different features. The "cross-architecture replication" narrative is constructed after the fact. Options:
- (a) Find a common feature that works on all three (z-scored features? stable_rank has lowest length confound)
- (b) Pre-register predictions for the next model tested
- (c) Reframe as "three single-model findings with a speculative unifying discussion" rather than "cross-architecture replication"

### Additional Items
- Bootstrap CIs on all per-group slopes (10K resamples)
- Test monotonicity: are slopes ORDERED by persona level, or just "different"?
- Report the non-persona control arm (Level 0.5) comparison explicitly — it's already in the data

---

## Concrete Spec for Missing Controls

### Control 1: Length-Matched Binning
```
For each model:
  1. Compute output length quartiles across all levels pooled
  2. Within each quartile, subset to prompts that fall in that bin for ALL 5 levels
  3. Compute per-level slopes on the length-matched subset
  4. Test: do slopes still differ between levels within matched bins?
```
This can be done on existing data — no new experiments needed.

### Control 2: Non-Persona Arm
New prompts needed (we can run on Beast):
```
V0: "You are a helpful assistant. Be brief. Use short sentences." (~70 tokens padded)
V1: "You are a helpful assistant. Provide moderate detail." (~70 tokens padded)  
V2: "You are a helpful assistant. Be thorough and detailed." (~70 tokens padded)
V3: "You are a helpful assistant. Be extremely comprehensive and exhaustive." (~70 tokens padded)
```
Same 50 user prompts, same models, same extraction. Compare verbosity slopes against persona slopes. If persona slopes are in a different feature or different direction from verbosity slopes, specificity is demonstrated.

### Control 3: Monotonicity Test
```
For the active feature on each model:
  Compute slope at each of the 5 levels
  Test: is the slope sequence monotonically ordered?
  Spearman correlation of (level_number, slope) with permutation test
```
Also on existing data.

---

## Recommended Paper Structure (Option A)

**Title:** "Persona Intensity Modulates Length-Geometry Scaling in Transformer KV-Cache: A Cross-Architecture Analysis"

**Lead with:** The interaction-term FWL methodology (the kill test) as a contribution. Show what pooled FWL misses.

**Structure:**
1. Introduction: Can persona be detected geometrically?
2. Methods: 5 levels, 3 models, delta features, FWL battery
3. Results Part 1: Linear FWL shows strong effects
4. Results Part 2: Interaction-term FWL kills static fingerprint
5. Results Part 3: Slope heterogeneity IS the signal
6. Results Part 4: Length-matched binning (does it survive?)
7. Results Part 5: Non-persona specificity control (is it persona-specific?)
8. Discussion: What does slope modulation mean? Architecture-specific features.
9. Limitations: Post-hoc feature selection, n=50, 3 architectures

**Key methodological contribution:** The interaction-term FWL diagnostic. This is useful beyond persona research — any KV-cache study with heterogeneous slopes needs this test.

---

## Data Location

All raw data on Beast at /home/thomas/oracle-harness-test/results/persona_intensity/:
- Qwen2.5-7B-Instruct_full.json (delta, valid)
- Llama-3.1-8B-Instruct_full.json (delta, valid, 6.8MB)
- Mistral-7B-Instruct-v0.3_full.json (delta, valid, latest run)

Qwen data summary in Muse repo: docs/persona-intensity-experiment/qwen_data_summary.json

---

## What We Can Do Now vs What Ang Needs To Do

**We can do (on Beast, no new experiments):**
- Length-matched binning analysis (Control 1) — existing data
- Monotonicity test (Control 3) — existing data
- Bootstrap CIs on slopes — existing data

**Ang needs to decide:**
- Non-persona control arm prompts (Control 2) — needs new prompt design, we run on Beast
- Framing choice: "cross-architecture replication" vs "three single-model findings"
- Whether to pursue a common feature (z-scored? stable_rank?)

**We run and red-team:**
- All new controls once designed
- The final paper draft

---

*The finding is real. The slope heterogeneity replicates across three architectures. But "slopes differ" needs to become "slopes differ specifically because of persona, not because of generic prompt variation." The specificity control is the bridge from observation to finding.*

*— Lyra, THCoalition Research*
