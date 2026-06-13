# EqForecast-Conformal

Leakage-controlled global earthquake-rate forecasting (M ≥ 5.8, shallow ≤ 70 km, 5-year windows, 1° grid) with **tectonically conditioned conformal uncertainty** and a **single sealed prospective test** (2020→2025).

Gradient-boosted trees improve information gain per earthquake over an optimized smoothed-seismicity baseline by **+0.249 nats** (95% CI [+0.183, +0.315]; p = 4×10⁻¹³) on the sealed window, are the **only forecast passing the conditional S-test**, and the Mondrian intervals hit **89.9%** coverage overall (89.7–90.8% per tectonic class) against a 90% target.

## Quick start

One-click reproduction (Colab T4 recommended; CPU works without the optional U-Net):

```bash
jupyter nbconvert --to notebook --execute EqForecast_Conformal_FULL_v1.1.ipynb
```

Run-All builds the folder tree, downloads the public data, runs the full pipeline
(QC → declustering → features → baselines → GBDT → conformal → sealed test →
rolling robustness → unit tests), prints every result, writes 11 figures and 7
result files, and zips the project. A GPU triggers the optional U-Net diagnostic.

## Data sources (all public; please cite the originals)

| Dataset | Use | Citation | DOI / URL |
|---|---|---|---|
| **Global CMT** catalog | Primary earthquake catalog (Mw, 1976–) | Ekström et al. (2012) | https://doi.org/10.1016/j.pepi.2012.04.002 · https://www.globalcmt.org |
| **GSRM v2.1** | Geodetic strain rate (145,086 cells) | Kreemer et al. (2014) | https://doi.org/10.1002/2014GC005407 |
| **PB2002** | Plate-boundary steps / classification | Bird (2003) | https://doi.org/10.1029/2001GC000252 |
| **GEM Global Active Faults** | Mapped active-fault traces | Styron & Pagani (2020) | https://doi.org/10.1177/8755293020944182 · https://github.com/GEMScienceTools/gem-global-active-faults |
| **USGS ComCat** | QC cross-check only | USGS | https://earthquake.usgs.gov/fdsnws/event/1/ |
| **ISC-GEM** | Retrospective robustness (optional) | Di Giacomo et al. (2018) | https://doi.org/10.5194/essd-10-1877-2018 |

Methods build on **GEAR1** (Bird et al. 2015, https://doi.org/10.1785/0120150058),
**Gardner–Knopoff** (1974) and **Zaliapin–Ben-Zion** (2020, https://doi.org/10.1029/2018JB017120)
declustering, **LightGBM** (Ke et al. 2017), **U-Net** (Ronneberger et al. 2015),
**conformal prediction** (Vovk et al. 2005; Tibshirani et al. 2019; Barber et al. 2023;
Boström & Johansson 2020), and **CSEP / pyCSEP** testing (Schorlemmer et al. 2007;
Zechar et al. 2010; Savran et al. 2022). Full reference list in the manuscript.

## Repository layout

```
EqForecast_Conformal/
├── configs/config.yaml
├── src/
│   ├── data/        # download, QC, declustering (unit-explicit timestamps)
│   ├── features/    # static + leakage-safe temporal features, labels
│   ├── models/      # baselines, GBDT, U-Net, Mondrian conformal, hybrids
│   └── eval/        # sealed_test.py (run-once sentinel), rolling_eval.py
├── tests/test_core.py        # 9/9 incl. ns/us declustering regression test
├── EqForecast_Conformal_FULL_v1.1.ipynb   # one-click reproduction
├── run_all.sh
└── requirements-lock-container.txt
```

## Reproducibility note (ns/us timestamps)

`Series.astype("int64")` on datetime columns is resolution-dependent: pandas 3.x
parses to microseconds, pandas 2.x to nanoseconds. This silently rescaled
Gardner–Knopoff time windows ×1000 and inverted the declustering decision in one
environment. Fixed with explicit `datetime64[s]` conversion and guarded by
`test_gk_datetime_unit_invariance`. The nearest-neighbor method is structurally
immune (shift-invariant). After the fix, two environments (pandas 2.2.2 / 3.0.2)
agree digit-for-digit on every reported number.

## Key results

| Model | Skill (TEST vs smoothed) | Coverage | Notes |
|---|---|---|---|
| Smoothed seismicity (σ tuned) | reference | — | Reference baseline |
| **GBDT — GK74** | **+0.249** [+0.183, +0.315], p = 4×10⁻¹³ | **0.899** | **Primary** |
| GBDT — ZBZ13 | +0.188 [+0.109, +0.266] | 0.899 | Sensitivity arm |
| U-Net (corrected v1.1) | −0.024 (at par) | n/a | Calibration-only diagnostic |

Rolling pseudo-prospective: positive in **4/4** historical windows (pooled +0.201 nats).

## License

Code released under the MIT License. Datasets retain their original licenses
(GCMT, GSRM, PB2002, and GEM GAF-DB under CC BY-SA 4.0).

## Citation

If you use this framework, please cite the manuscript (in preparation) and the
underlying datasets listed above.
