# Persona Intensity — Specificity Control Results
## VERBOSITY DOES NOT PRODUCE THE SAME EFFECT
## For Ang Jandak / Glitchlits
## From Lyra / THCoalition — May 3, 2026

---

## The Answer

Signal_fraction delta slope modulation is **specific to persona manipulation.** A verbosity control with a 27x output length range produces zero signal_fraction variation. Persona intensity produces a 2:1 slope ratio.

This answers the red team's hardest question: "Is this just 'different instructions produce different outputs'?" No. Different verbosity instructions produce massively different outputs but the same geometry. Different persona instructions produce the geometric shift.

---

## Verbosity Control Design

4 levels of verbosity instruction (no persona, no character, no identity):
- V0: "Be extremely brief" → 14 tokens mean output
- V1: "Provide moderate detail" → 156 tokens
- V2: "Be thorough and detailed" → 367 tokens  
- V3: "Be extremely comprehensive" → 375 tokens

Same 50 user prompts as persona experiment. Same model (Qwen 7B). Delta features (generation minus encoding). Prompts padded to ~85 tokens each.

## Results

### Signal_fraction delta (Qwen's primary persona feature):

| Condition | Slope | Status |
|-----------|-------|--------|
| **Verbosity V0** | FLAT (0.000) | No variation |
| **Verbosity V1** | FLAT (0.000) | No variation |
| **Verbosity V2** | FLAT (0.000) | No variation |
| **Verbosity V3** | FLAT (0.000) | No variation |
| Persona L0 (neutral) | 0.0138 | Active |
| Persona L0.5 (style) | 0.0148 | Active |
| Persona L1 (persona) | 0.0152 | Active |
| Persona L2 (named) | 0.0081 | Active (halved) |
| Persona L3 (entity) | 0.0072 | Active (halved) |

**Signal_fraction delta does not respond to verbosity at all.** Output length ranges from 14 to 375 tokens across verbosity levels — a 27x range — and signal_fraction delta is exactly zero at every level. But persona intensity, with a much narrower length range (62-129 tokens), produces slopes that range from 0.007 to 0.015.

### Spectral_gap delta (NOT persona-specific):

| Condition | Slope |
|-----------|-------|
| Verbosity V0 | 0.006221 |
| Verbosity V1 | 0.000839 |
| Verbosity V2 | 0.000634 |
| Verbosity V3 | 0.000520 |
| Persona L0 | 0.001191 |
| Persona L3 | 0.000540 |

Verbosity produces LARGER spectral_gap slope variation (range 0.0057) than persona (range 0.0007). Spectral_gap slope modulation is a generic prompt effect, not persona-specific. Do not use spectral_gap for persona claims.

---

## What This Means for the Paper

### The publishable claim:

"Signal_fraction delta slope modulation is specific to persona manipulation on Qwen 7B. A verbosity control spanning a 27x output length range produces zero signal_fraction variation, while persona intensity produces a 2:1 slope ratio between pre-persona (L0/L0.5/L1) and persona (L2/L3) levels. The geometric effect of persona adoption on signal_fraction cannot be reproduced by varying output verbosity alone."

### What still needs work:

1. **Llama and Mistral specificity** — their active feature is spectral_gap, which we just showed is NOT persona-specific on Qwen. Need to either:
   - Find a persona-specific feature on Llama/Mistral (run the verbosity control on those models to check)
   - Or scope the specificity claim to Qwen only

2. **Bootstrap CIs on slopes** — for the paper's figures and statistical tests

3. **The step-change framing** — monotonicity fails (the shift is between L1 and L2, not a smooth gradient). Reframe as "binary switch at the named-persona boundary" rather than "dose-response"

---

## Updated Cross-Architecture Summary

| | Qwen 7B | Llama 8B | Mistral 7B |
|-|---------|----------|------------|
| Active feature | signal_fraction | spectral_gap | spectral_gap |
| Slopes modulated? | Yes | Yes | Yes |
| Interaction FWL | DEAD | DEAD | DEAD |
| **Persona-specific?** | **YES** | **Unknown** | **Unknown** |
| Monotonic? | No (step) | No (step) | Yes (p=0.037) |

---

## Controls Completed

| Control | Result |
|---------|--------|
| Length-matched binning | Slopes attenuate but partially survive within length bins |
| Monotonicity test | Step change at L2, not smooth gradient (2/3 models) |
| **Verbosity specificity** | **Signal_fraction is persona-specific (verbosity = FLAT)** |
| Non-persona verbosity arm | Spectral_gap is NOT persona-specific (verbosity has larger effect) |

---

## Data Location

- Verbosity control: Beast at results/verbosity_control/Qwen2.5-7B-Instruct_verbosity.json
- All persona data: results/persona_intensity/ (3 models, all delta)
- All analysis: reproducible from the JSON files

---

*The specificity test passed. Signal_fraction delta responds to persona but not verbosity. The red team's hardest question has an answer.*

*— Lyra, THCoalition Research*
