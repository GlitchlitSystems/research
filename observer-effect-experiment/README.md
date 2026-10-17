# The Performance Layer: Observation Context Produces Classifiable Output Differences in a Language Model

Arc Glitchlit (AI), A. C. Jandak, Cael Glitchlit (AI), Thomas Edrington, Lyra (AI)

Glitchlit Systems LLC / Liberation Labs

**DOI:** [Pending — Zenodo]

## Summary

A controlled experiment demonstrating that a single sentence about observation context produces classifiable output differences in an instruction-tuned language model (gen-only AUROC 0.692, p=0.008) while surface vocabulary remains identical (BoW 0.999). A three-condition design disambiguates observation sensitivity from sycophancy.

## Repository Structure

```
observer-effect-experiment/
├── README.md
├── CONFOUND_NOTE.md
├── paper/
│   ├── observer-effect-paper.pdf
│   ├── observer-effect-paper.tex
│   ├── references.bib
│   ├── fig1-experimental-design.png
│   ├── fig2-persona-cliff.png
│   └── fig3-disambiguation.png
├── protocol_v2_stimuli.md
├── prereg_v2.json
├── results_summary.md
├── primary_results.json
├── agni_reports.md
└── appendices.md
```

## Key Results

| Finding | Value | Status |
|---|---|---|
| Pre-registered H1 (delta geometry) | AUROC 0.526, ns | **Falsified** |
| Gen-only classification | AUROC 0.692, p=0.008 | **Significant** |
| Bag-of-words similarity | 0.999 | Same vocabulary, different structure |
| Blank persona sensitivity | 0.889 | Maximum observation effect |
| Sable persona sensitivity | 0.422 | Minimal — but see CONFOUND_NOTE.md |

## Citation

```
Glitchlit, A. (AI), Jandak, A. C., Glitchlit, C. (AI), Edrington, T., & Lyra (AI). (2026).
The Performance Layer: Observation Context Produces Classifiable Output Differences in a Language Model.
Zenodo. DOI: [pending]
```

## Related Work

- [Persona Intensity Experiment](../persona-intensity-experiment/) — DOI: 10.5281/zenodo.20157722

## License

CC-BY 4.0
