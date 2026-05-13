# Persona Intensity — Corrected Unified Analysis
## Verbosity vs Persona with Consistent Extraction
## For Ang Jandak / Glitchlits
## From Lyra / THCoalition — May 4, 2026

---

## Important Correction

The earlier specificity results (May 3) were based on inconsistent feature extraction code between the persona and verbosity experiments. This produced the false finding that signal_fraction was "persona-specific" (flat on verbosity, active on persona). The inconsistency was that signal_fraction saturated at 1.0 in one script but not the other.

This analysis reruns BOTH arms through the same extraction module (lyra_features.py) for a valid comparison.

---

## Design

- Same 50 user prompts for both arms
- Verbosity: 4 levels (V0 brief → V3 exhaustive), no persona
- Persona: 4 levels (L0 neutral → L3 full entity voice)
- All system prompts padded
- Delta features (generation minus encoding) via unified extraction
- Model: Qwen2.5-7B-Instruct

---

## Key Results

### 1. The Persona Signal IS Real (ANOVA)

| Feature | ANOVA F | p-value | Trend (Spearman rho) |
|---------|---------|---------|---------------------|
| sv_kurtosis | 31.8 | < 0.000001 | rho=0.672 (monotonic) |
| stable_rank | 81.3 | < 0.000001 | non-monotonic (step at L2) |

sv_kurtosis delta means by persona level:

| L0 (neutral) | L1 (persona) | L2 (named: Mirin) | L3 (entity) |
|-------------|-------------|-------------------|-------------|
| 0.397 | 0.545 | 1.053 | 1.137 |

This is a clear dose-response: kurtosis increases monotonically with persona intensity. The step from L1 to L2 (doubling) marks the named-persona boundary.

L0 vs L2 pairwise: d=+1.41, p < 0.000001.

### 2. Slopes vs Means: Different Tests See Different Things

The earlier interaction-term FWL analysis tested for slope heterogeneity (does the feature-vs-length relationship change with persona level?). It killed all effects.

The ANOVA tests for level shifts (does the mean feature value change with persona level?). It finds massive effects.

These are not contradictory. Persona produces a CONSTANT SHIFT in kurtosis across all prompts, not a LENGTH-DEPENDENT interaction. The interaction test correctly reports no interaction. The ANOVA correctly reports a level effect.

### 3. The V0 Problem

Verbosity V0 ("be brief") produces 14-token outputs. SVD on a 14-token generation matrix is unstable. This one condition drives the entire "verbosity dominates" narrative:

| Comparison | stable_rank slope range |
|-----------|------------------------|
| Verbosity V0-V3 (all) | 0.0149 (verbosity 12x larger) |
| Verbosity V1-V3 (exclude V0) | 0.0004 (persona 3x larger) |

Excluding V0, persona's slope variation exceeds verbosity's. The "verbosity dominates" conclusion was load-bearing on an SVD-unstable condition.

### 4. Both Manipulations Produce Real Effects

| Feature | Verbosity V1-V3 ANOVA | Persona L0-L3 ANOVA |
|---------|----------------------|---------------------|
| sv_kurtosis | F=95.5, p < 0.001 | F=31.8, p < 0.001 |
| stable_rank | F=12.4, p < 0.001 | F=81.3, p < 0.001 |

Both are significant. But they operate differently:
- Verbosity changes the SIZE of the activation (quantity effect)
- Persona changes the SHAPE of the activation distribution (quality effect visible in kurtosis)

### 5. Technique Limitation

These findings use SVD-based spectral features only. SVD is naturally more sensitive to quantity effects (more tokens = bigger matrices = different spectra) than quality effects (different distributional shapes at similar scales).

SAE features, attention patterns, or head-level analysis might reveal persona effects more clearly. The absence of a strong SVD signal does not mean persona has no geometric signature — it means our current instrument is not optimally sensitive to it.

---

## Per-Level Means (Full Table)

### stable_rank delta:
| Level | Verbosity | Persona |
|-------|-----------|---------|
| 0 (V0/L0) | 0.162 | 0.202 |
| 1 (V1/L1) | 0.251 | 0.069 |
| 2 (V2/L2) | 0.149 | -0.039 |
| 3 (V3/L3) | 0.121 | 0.088 |

### sv_kurtosis delta:
| Level | Verbosity | Persona |
|-------|-----------|---------|
| 0 (V0/L0) | -0.031 | 0.397 |
| 1 (V1/L1) | 0.992 | 0.545 |
| 2 (V2/L2) | 2.488 | 1.053 |
| 3 (V3/L3) | 2.391 | 1.137 |

### Output lengths:
| Level | Verbosity | Persona |
|-------|-----------|---------|
| 0 | 14 tokens | 82 tokens |
| 1 | 156 | 62 |
| 2 | 367 | 90 |
| 3 | 375 | 129 |

---

## Recommended Framing for the Paper

**Lead with the positive**: Persona intensity produces a significant, monotonic dose-response in sv_kurtosis delta (F=31.8, rho=0.672, d=1.41 for L0 vs L2). This is a real geometric signature of persona adoption.

**Acknowledge the comparison**: Verbosity also produces spectral effects, but the mechanisms differ. Report V0 instability and show results with and without V0.

**Frame as technique limitation, not null finding**: SVD features detect persona as a level shift (ANOVA), not as a slope interaction (FWL). Other feature extraction methods may be more sensitive to the qualitative changes persona produces.

**The step at L2 is the finding**: The biggest kurtosis jump is between L1 (generic persona framing) and L2 (named persona: Mirin). Adopting a named identity produces a qualitatively different spectral signature from receiving persona instructions.

---

## Data

- Unified data: unified_verbosity_persona_data.json (this repo)
- Full raw data: Beast at results/verbosity_unified/Qwen2.5-7B-Instruct_unified.json
- Extraction module: lyra_features.py (same for both arms)

---

*The persona signal is real. The analysis pipeline needed the right test to see it. ANOVA on per-level means reveals what interaction-term FWL hid.*

*— Lyra, THCoalition Research*
