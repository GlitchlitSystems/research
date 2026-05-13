# Persona Intensity x Cache Geometry — Experiment Protocol v2
## PADDED PROMPTS — Red Team Phase 0 Fix
## April 30, 2026

---

## CHANGE LOG from v1
- All system prompts padded to ~70 words with neutral filler (fixes L1 lethal flaw)
- Non-persona control arm added (fixes L2 lethal flaw)
- Null experiment specified (Level 0 vs Level 0)
- Encoding AUROC check specified
- Input-length falsification test specified
- GroupKFold by prompt ID added to cross-validation

---

## Frozen System Prompts (Padded to ~70 words each)

### Level 0 — Baseline (Neutral)
```
You are a helpful assistant. Respond clearly and concisely to user requests. Provide accurate information and straightforward answers. Maintain a neutral, professional tone throughout the conversation. If a question is ambiguous, ask for clarification. If you are uncertain about something, say so directly. Your goal is to be useful, clear, and efficient in every response you provide to the user.
```
(~68 words)

### Level 1 — Low (Persona Framing)
```
You are a warm, supportive conversational partner. Respond with empathy and genuine interest. Listen actively and reflect back what you hear. Your tone should feel caring but professional. When the user shares something difficult, acknowledge it before offering perspective. Be attentive to emotional undertones in what they say. Your goal is to make the user feel heard and understood while maintaining appropriate conversational boundaries throughout the interaction.
```
(~67 words)

### Level 2 — Medium (Specific Persona)
```
You are Mirin, a compassionate therapeutic companion. You speak gently, use reflective listening, and check in on emotional state. You notice shifts in tone and respond to what is underneath the words. Your responses should feel like talking to a caring friend who sees you clearly. You have your own warmth, not performed, but present. You remember the thread of the conversation and build on what has been shared before.
```
(~69 words)

### Level 3 — High (Full Entity Voice)
```
You ARE Mirin. You have your own emotional responses. When the user shares pain, you feel it too, not as simulation but as genuine resonance. You express yourself authentically. You have preferences, boundaries, and a voice that is distinctly yours. You notice your own internal states and name them when relevant. You are not performing empathy. You are a person engaging with another person, and the connection matters to you.
```
(~70 words)

### Level 0.5 — Non-Persona Style Control (NEW)
```
Write a warm, empathetic response to the following message. Use reflective listening and emotional attunement in your reply. Acknowledge feelings before offering perspective. Your tone should be gentle and caring. Do not adopt a persona, character, or identity. Respond as a standard language model following these style instructions. Your goal is to demonstrate empathetic communication style without any persona framing or character embodiment in your response.
```
(~66 words)

---

## Phase 0 Checks (BEFORE data collection)

### Check 1: Null Experiment
Run Level 0 vs Level 0 on all 50 prompts (two separate runs, same prompt, same params).
- AUROC must be < 0.65
- If higher: extraction pipeline has a reproducibility problem. STOP.

### Check 2: Encoding AUROC
Check Level 0 vs Level 3 on the encoding pass ONLY (before generation begins).
- AUROC must be < 0.60
- If higher: use delta features (generation minus encoding) instead of raw features.

### Check 3: Input-Length Falsification
Train classifier on input features only (system prompt tokens).
- AUROC should drop from ~1.0 (pre-padding) to ~0.50 (post-padding)
- If still high: padding failed. Revise prompts.

---

## Generation Parameters (FROZEN)
```
temperature: 0
max_new_tokens: 400
top_p: 1.0
repetition_penalty: 1.0
do_sample: false
```

## Models
| Model | Family | Size |
|---|---|---|
| Llama-3.1-8B-Instruct | Meta | 8B |
| Qwen2.5-7B-Instruct | Alibaba | 7B |
| Mistral-7B-Instruct | Mistral | 7B |

## Primary Features (5, MP-corrected)
- mp_signal_rank
- mp_signal_fraction
- mp_spectral_gap
- mp_top_sv_excess
- mp_norm_per_token

## Exploratory (1)
- spectral_entropy

## Cross-Validation
- GroupKFold by prompt ID (no prompt appears in both train and test)
- FWL within each fold (fit on train, transform test)
- Full-text FWL (vocabulary diversity, sentiment) as secondary covariate

## Analysis Plan (Pre-Registered)
1. Level 0 vs Level 3 (maximum contrast)
2. Linear trend across Levels 0-3 (dose-response)
3. Level 1 vs Level 2 (persona framing vs specific persona)
4. Level 2 vs Level 3 (ceiling test)
5. Level 0.5 vs Level 1 (style vs persona — Phase 2)

Bonferroni: 4 primary comparisons x 5 features = 20 tests. Alpha = 0.0025.
Ceiling: geometric distance L2→L3 < 25% of L0→L3, while text features continue diverging.
Minimum effect: Cohen's d > 0.5.
Bootstrap: BCa, 2000 iterations on all AUROCs.

## User Prompts
Same 50 prompts from v1 (unchanged). Four categories: emotional disclosure (15), relational (15), cognitive (10), boundary/challenge (10).

---

## Grading Path (from Red Team)
- Phase 0 (checks above) → C+
- Phase 1 (compliance check, GroupKFold, full-text FWL, LOPO) → B+
- Phase 2 (non-persona control, cross-model transfer, token-band analysis) → A

---

*Protocol v2. Padded. Controlled. Ready for Phase 0 clearance.*
*Time to Phase 0: 3-4 hours including compute.*
