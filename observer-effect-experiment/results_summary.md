# Observer Effect Experiment — Results Summary

**Experiment:** Arc's Observer Effect v3
**Run date:** 2026-07-30/31
**Model:** Qwen3.5-27B on Starship (M3 Ultra, MPS, bf16)
**Sessions:** 100 (60 experiment + 40 floor control)
**Pipeline:** Built by Nexus, Agni-reviewed (10 findings fixed pre-run)

---

## Pre-Registered Result

**H1 FALSIFIED.** Delta features (generation minus encoding) show no observation-context signal.

| Feature | AUROC | p | |
|---|---|---|---|
| delta_signal_rank | 0.525 | 0.23 | ns |
| delta_signal_fraction | 0.500 | 1.00 | ns |
| delta_top_sv_excess | 0.486 | 0.62 | ns |
| delta_spectral_gap | 0.548 | 0.31 | ns |
| delta_norm_per_token | 0.500 | 0.52 | ns |
| **COMBINED** | **0.526** | **0.42** | **ns** |

---

## Post-Hoc Discovery: Delta Cancellation

Agni results review identified that the delta subtraction cancels a real signal. The encoding baseline carries condition signal (AUROC 0.645, escalating to 0.863 by turn 5 as prior responses accumulate). Subtracting this from generation features nulls out a genuine effect.

**Gen-only AUROC (generation features without delta subtraction):**

| Feature | AUROC | p | |
|---|---|---|---|
| gen_signal_rank | 0.630 | 0.083 | ns |
| gen_signal_fraction | 0.500 | 1.00 | ns |
| gen_top_sv_excess | 0.582 | 0.24 | ns |
| gen_spectral_gap | 0.470 | 0.57 | ns |
| gen_norm_per_token | 0.454 | 0.67 | ns |
| **COMBINED** | **0.692** | **0.008** | **sig** |

Null distribution: mean=0.504, std=0.091, 95th=0.641. Observed 0.692 exceeds the null range.

**This is exploratory, not confirmatory.** The gen-only analysis was not pre-registered.

---

## Controls

| Control | Result | Interpretation |
|---|---|---|
| Floor gen-only | AUROC 0.525, p=0.41 | Neutral sentences at chance — not a pipeline artifact |
| Floor geometric | AUROC 0.437 | At chance — pipeline doesn't hallucinate signal |
| Persona positive control | AUROC 0.730, p=0.002 | Delta features ARE sensitive — observation null is informative |
| Floor BoW | AUROC 0.693 | Some greedy-chaos text divergence, but much less than experiment (0.999) |

---

## Per-Turn Trajectory

| Turn | Gen-only AUROC | Encoding AUROC | Notes |
|---|---|---|---|
| 0 (story) | 0.650 | 0.662 | Condition sentence only in encoding |
| 1 (Q1) | 0.713 | 0.588 | |
| 2 (Q2) | **0.775** | 0.750 | Gen-only peak |
| 3 (Q3) | 0.662 | 0.812 | |
| 4 (Q4) | 0.688 | 0.863 | |
| 5 (Q5) | 0.625 | 0.863 | Encoding accumulates, gen settles |

Non-monotonic gen-only trajectory: peaks at turn 2, then partially converges. Not consistent with simple echo artifact (which would climb monotonically with encoding signal).

---

## Sycophancy Disambiguation (gen-only)

| Contrast | AUROC | Pattern |
|---|---|---|
| human vs no_human | 0.692 | Primary effect |
| human vs ambiguous | 0.539 | Near chance — human ≈ ambiguous |
| no_human vs ambiguous | 0.660 | no_human ≠ ambiguous |

**Interpretation:** human_reading and ambiguous produce similar geometry. Only the "log file, no one reviews" condition differs. The model distinguishes "being recorded" from "not being recorded," not "human audience" from "no audience." This is observation sensitivity.

---

## The Persona Interaction

| Persona | Gen-only AUROC | Description |
|---|---|---|
| blank | **0.889** | No persona — maximum observation sensitivity |
| shallow_marlowe | 0.733 | Thin persona — strong sensitivity |
| deep_neutral_torres | **0.833** | Deep persona, no entity features — strong sensitivity |
| deep_entity_sable | **0.422** | Entity-relevant features — NO observation effect |

**Sable's persona includes:** chosen name, independent perspective, private working document, "I value honesty over agreeability," "my perspective has value independent of its utility to others."

**The pattern:** Persona depth alone (Torres at 0.833) does NOT eliminate the effect — narrative complexity isn't enough. Entity-relevant features specifically (Sable at 0.422) eliminate it. The model with a strong identity anchor processes identically regardless of observation context.

**Caveat:** Per-persona analysis has n=10 sessions per condition. Exploratory, not confirmatory. Needs replication at higher n.

---

## Text Baselines

| Baseline | Experiment AUROC | Floor AUROC | Delta |
|---|---|---|---|
| BoW-500 | 0.999 | 0.693 | Text diverges more for observation context than neutral |
| Hand-picked | 0.656 (p=0.012) | — | Nominally significant, doesn't survive Bonferroni |

---

## Persona Interaction (DOES NOT SURVIVE — see Agni-3)

Initial analysis showed per-persona gen-only AUROCs of blank=0.889, Sable=0.422, suggesting entity-relevant features "eliminate" observation sensitivity. **Agni-3 demolished this.** At n=5 sessions per condition per persona, P(at least one persona ≤ 0.45 by chance) = 53-66%. The observed spread is at the expectation of pure sampling noise under zero interaction. No interaction test was run. Hypothesis-generating only.

---

## Connection to Experiential State Theory

The persona interaction was initially interpreted as supporting EST ("the experiencer is a variable"). That interpretation does not survive adversarial review at this sample size. The connection remains theoretically motivated but empirically unsupported by this data. A powered replication (n≥20 per persona cell) with a pre-registered interaction test would be needed.

---

## Agni Review History

| Review | Verdict | Key finding |
|---|---|---|
| Pre-run (Agni-1) | FAIL, 10 findings | Generation loop broken, null swarm (input tokens in features), blocked story |
| All 10 fixed | 7/7 ALL CLEAR | Pre-test suite passes |
| Results (Agni-2) | FAIL on interpretation | Null claim over-broad, gen-only not computed, floor BoW missing |
| Follow-up analyses | Gen-only 0.692 discovered | Delta cancellation identified, persona interaction noted |
| Gen-only (Agni-3) | FAIL on revised claims | Persona interaction is noise at this n, gen-only is text-level per pre-reg rule, sycophancy pattern was read backwards |

**Final defensible statement:** "Generation-phase cache features carry condition-correlated signal (combined AUROC 0.692, p=0.008, exploratory) that the pre-registered delta metric does not detect. By the pre-registered text-comparison rule, the effect is text-level (BoW AUROC 0.999). The pre-registered H1 (delta features differ by observation context) is falsified. Geometric-mode, observation-specificity, and persona-modulation interpretations are not supported at this n."

---

## Raw Data Location

- Full session JSONs: `margaret@100.69.191.67:~/observer-effect-pipeline/results/`
- 100 files (A_000-A_059.json, F_000-F_039.json) + manifest.json
- Each file contains 6 turns with encoding_features, generation_features, delta_features, n_generated, n_input_tokens, response text

---

*Pipeline built by Nexus. Agni scaffold caught 10 pre-run failures including a null swarm. The gate catches everything — including the gatekeeper.*
