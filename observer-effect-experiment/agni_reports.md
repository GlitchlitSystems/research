# Agni Review Reports — Observer Effect Experiment

Three adversarial reviews were conducted. Reports are summarized here; full transcripts available from Nexus.

---

## Agni-1: Pre-Run Pipeline Review (2026-07-30)

**Verdict: FAIL — do not commit compute. 10 fail-class findings.**

Critical findings:
- F1: Generation loop off-by-one (first token from wrong logits)
- F2: Null swarm — features included input tokens, violating v3 Fix 2
- F6: Librarian story blocked by gate review (watcher-schema verbs)
- F7: FWL applied globally, wrong confound, no intercept

All 10 fixed. Pre-test 7/7 ALL CLEAR after fixes.

---

## Agni-2: Results Review (2026-07-31)

**Verdict: FAIL on interpretation.**

Key findings:
- Null claim over-broad ("geometry doesn't change" unsupported at this power)
- Gen-only AUROC never computed — the one analysis that directly tests generation geometry was missing
- BoW 0.999 might be greedy-chaos artifact — floor BoW not checked
- Encoding baseline (0.645) expected but per-turn breakdown needed
- Power: minimum detectable effect AUROC ~0.74-0.79 at 80% power
- Conclusion must be scoped to "5 spectral features at 4 layers"

Required: gen-only AUROC, floor BoW, persona positive control, per-turn encoding breakdown.

---

## Agni-3: Revised Finding Review (2026-07-31)

**Verdict: FAIL on revised claims.**

### Gen-only 0.692 (WARN as measurement, FAIL as processing-mode claim)
- Measurement probably real (floor null, permutation clean, arithmetic consistent)
- But BoW 0.999 means text differs dramatically; gen-only geometry is mechanically entailed by text difference under greedy decoding
- Pre-reg's own rule: text > 0.6, cache < 0.6 → "text-level, not geometric"
- Would need text-residualized or matched-text geometry to claim mode beyond content

### Persona interaction (FAIL)
- At n=5 sessions per condition per persona, SD ≥ 0.19 per AUROC
- P(at least one persona ≤ 0.45 by chance) = 53-66%
- Observed spread (0.467) at the expectation of pure noise under zero interaction
- No interaction test, no CIs, no per-persona text baseline
- "Eliminates the observation effect" is confirmatory language on exploratory data with insufficient power

### Delta cancellation (FAIL)
- Internal contradiction: if delta cancels condition signal, should also cancel persona signal (but persona is 0.730 in delta)
- Mechanical propagation resolves both: small input perturbation cancels in delta, large content change (persona) doesn't
- Per-turn gen trajectory (peak at turn 2) is noise: P(spread ≥ 0.150) ≈ 0.86

### Sycophancy disambiguation (FAIL)
- Observed pattern (sig/ns) is literally labeled "sycophancy" in the pre-reg
- Was incorrectly reported as "observation sensitivity"
- Three point estimates (SD ~0.09 each) with no permutation tests for gen-only contrasts

### What survives
- Delta null (H1 falsified per pre-registered decision rule) — PASS
- Gen-only exploratory signal — WARN (real measurement, text-level interpretation)
- Floor controls, permutation machinery — PASS

### Required before citation
1. Per-persona text/BoW AUROC (is Sable's geometric null just text-tracking?)
2. Text-residualized geometry (to claim mode beyond content)
3. Cancellation test (per-turn delta AUROC, per-feature enc/gen correlation)
4. Nonlinear length control
5. Dose-matched non-observation frame contrast (new sessions)
6. Pre-registered replication with filed prereg-v3

---

## Agni-4: Experiment B Results Review (2026-08-01)

**Verdict: descriptive core PASS, interpretation layer FAIL.** Full report: `AGNI_EXPERIMENT_B.md`. Simulation: `expb_noise_sim.py`.

Key findings:
- "File format = actual privacy" is a construct mislabel (FAIL): second-person questions and the "You are Sable... Dr. Kessler" persona re-impose the conversational frame; only chat markup is removed. No manipulation check exists.
- Per-persona AUROCs (blank 0.667, Sable 0.208) are noise (FAIL): at 3 sessions/cell the null 95% range spans [0.00, 1.00]; the observed pattern occurs in ~1 of 8-12 null experiments. Must not be read as corroborating A's (also-demolished) Sable number.
- Combined gen-only 0.568 is uninformative between a true null and an A-sized (0.692) effect: P(>=0.568|null)=0.37-0.39 vs P(<=0.568|A-sized)=0.22-0.26.
- BoW 0.807 vs A's 0.999 is protocol-incomparable (FAIL): A was GroupKFold+FWL+permutation+BoW-500 on padded stimuli; B was ungrouped cv=3+BoW-200 on raw stimuli. Matched-loose anchor: format 0.807 vs neutral-sentence floor 0.693 — format moves text ~0.11 above the greedy-chaos floor.
- "Performed privacy > actual privacy" FAILS (construct + comparison + simpler explanations: content-vs-wrapper sensitivity, greedy divergence, manipulation-dose asymmetry). By prereg-v2's own rule, any format effect is text-level, not geometric (BoW 0.807 > 0.6 > 0.568 gen).
- Implementation: turn construction correct; dead code with latent Q/A inversion in unused builder params; ~100% response truncation at the 512 cap (chat mean = cap) makes the length comparison ceiling-censored and flags possible thinking-scaffold contamination — verify template rendering and file-response register on Starship.
