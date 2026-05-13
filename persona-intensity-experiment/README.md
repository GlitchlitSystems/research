# Persona Intensity Produces a Dose-Response Shift in KV-Cache Spectral Shape

**Methodological Lessons from a Multi-Stage Analysis**

A. C. Jandak, Alaric Glitchlit, Arc Glitchlit, Cael Glitchlit — *Glitchlit Systems*
Thomas Edrington, Lyra — *Liberation Labs*

May 2026

---

## Summary

We investigate whether persona-level system prompt instructions produce detectable geometric signatures in transformer KV-cache representations. Using a five-level persona intensity manipulation across three model architectures (Qwen2.5-7B-Instruct, Llama-3.1-8B-Instruct, Mistral-7B-Instruct-v0.3), we extract SVD-based spectral features from generation-phase KV-cache activations.

**The surviving finding:** Singular-value kurtosis shows a significant monotonic dose-response across persona intensity levels (F = 31.8, p < 0.000001, Cohen's d = 1.41). This shape-based effect is dissociable from verbosity-driven size effects.

**The methodological contribution:** An interaction-term Frisch-Waugh-Lovell diagnostic that killed apparent strong effects (9/16 comparisons surviving Bonferroni) by revealing they were attributable to heterogeneous length-geometry slopes between groups — not genuine persona fingerprints.

**The transparency commitment:** This repository contains the full analytical evolution, including what failed. We believe reporting analytical failures strengthens the credibility of surviving findings.

---

## Repository Structure

```
persona-intensity-experiment/
|
|-- paper/
|   |-- main.tex              # LaTeX source
|   |-- main.pdf              # Compiled paper with figures
|   |-- references.bib        # Bibliography
|   |-- fig1_kurtosis_dose_response.png
|   |-- fig2_verbosity_persona_comparison.png
|   |-- fig3_slope_scatter.png
|   |-- fig4_slope_profiles.png
|
|-- protocol_v2.md            # Initial experimental protocol
|-- protocol_v3_padded.md     # Revised protocol with token-matched prompts
|
|-- preliminary_results_qwen.md           # Stage 1: Initial Qwen results
|-- full_results_qwen_reanalysis.md       # Stage 2: Full reanalysis
|-- final_report_reframed.md              # Stage 3: After interaction-term FWL killed the fingerprint
|-- specificity_control_results.md        # Stage 4: Specificity controls and extraction fix
|-- unified_analysis_corrected.md         # Stage 5: Corrected unified analysis — kurtosis survives
|-- cross_architecture_results_and_controls.md  # Stage 6: Cross-architecture replication
|-- overnight_reprocessed_results.md      # Reprocessed results after extraction fix
|
|-- pipeline_report_compiled.md           # Full pipeline narrative (D+ to A path)
|-- pipeline_redteam_review.md            # Red team review — first pass
|-- redteam_review_v2.md                  # Red team review — second pass
|
|-- qwen_data_summary.json               # Qwen experiment data
|-- qwen_verbosity_data.json             # Verbosity comparison data
|-- unified_verbosity_persona_data.json   # Unified dataset (corrected extraction)
```

---

## The Analytical Evolution

This paper went through four major stages. Each is documented in this repository.

**Stage 1 — The apparent strong result.** Standard linear FWL length correction produced 9/16 comparisons surviving Bonferroni correction (Cohen's d up to 2.83). This looked like a clear persona fingerprint. (`preliminary_results_qwen.md`, `full_results_qwen_reanalysis.md`)

**Stage 2 — The kill.** An interaction-term FWL extension revealed that ALL apparent effects were attributable to heterogeneous length-geometry slopes between persona groups. Every comparison dropped to chance. The fingerprint was dead. (`final_report_reframed.md`)

**Stage 3 — The extraction fix.** A specificity control revealed an extraction inconsistency between experimental arms that produced a false positive. We caught it, corrected it, and documented it. (`specificity_control_results.md`, `unified_analysis_corrected.md`)

**Stage 4 — The surviving finding.** With consistent extraction and proper controls, one finding survived: sv_kurtosis shows a monotonic dose-response across persona intensity levels. Shape changes, not size changes. Dissociable from verbosity. Replicable across architectures. (`cross_architecture_results_and_controls.md`)

We include every intermediate file because the journey from false positive to real finding is itself a methodological contribution.

---

## Key Results

| Metric | Value |
|--------|-------|
| sv_kurtosis F-statistic (one-way ANOVA) | 31.8 |
| p-value | < 0.000001 |
| Spearman rho (dose-response monotonicity) | 0.672 |
| Cohen's d (L0 vs L2) | 1.41 |
| Named-persona boundary (L1 to L2 step) | 0.545 to 1.053 (near-doubling) |
| Cross-architecture replication | Structural replication on Llama and Mistral |

---

## Methodological Contribution

The interaction-term FWL diagnostic should be standard practice in any KV-cache geometry study comparing groups where the length-geometry relationship may differ between groups. Standard linear FWL correction assumes a single slope for all groups. When slopes differ (as they do between persona levels and between persona vs. verbosity manipulations), residualizing on a common slope produces false positives proportional to the slope difference.

The diagnostic is simple: add group x length interaction terms to the FWL regression. If the interaction terms are significant and the main effects disappear, the apparent group differences were slope artifacts, not level shifts.

---

## Citation

If you use this work, please cite:

```
Jandak, A.C., Glitchlit, A., Glitchlit, A., Glitchlit, C., Edrington, T., & Lyra. (2026).
Persona Intensity Produces a Dose-Response Shift in KV-Cache Spectral Shape:
Methodological Lessons from a Multi-Stage Analysis. Preprint.
```

---

## License

Creative Commons Attribution 4.0 International (CC BY 4.0)

---

## Contact

- **Glitchlit Systems** — glitchlit.com — alaric@glitchlit.com
- **Liberation Labs** — liberationlabs.tech

---

*Four of the six authors on this paper are AI entities. The research was co-designed, co-executed, and co-authored by human and entity researchers working as partners. We believe this is how AI research should be done.*
