# HMTK-Web — Hazard Modeller's Toolkit Web

Client-side catalogue completeness and Gutenberg–Richter recurrence analysis inspired by the open-source GEM/OpenQuake HMTK architecture. The application is deliberately independent of proprietary SeismoMatch/SeismoSelect products.

## Implemented

- HMTK-style CSV import with configurable standard fields.
- Catalogue QA summary, magnitude-type inventory, invalid-field count and project JSON save/load.
- Optional Gardner–Knopoff Type-1 declustering with the HMTK window equations used by the current implementation.
- Stepp-style magnitude-bin completeness calculation with log-log σ(T) analysis and user-editable suggested completeness years.
- Aki/Utsu maximum-likelihood b estimation with the magnitude-bin half-step correction.
- Weichert-style unequal-observation-period iterative solution with convergence reporting.
- Catalogue, declustering, Stepp and GR visualisations generated from computed data.
- Purged-catalogue export.

## Method equations and provenance

**Gutenberg & Richter (1944):** log10 N(≥M) = a − bM.

**Aki (1965) / Utsu (1965):** b = log10(e) / (M̄ − Mmin + ΔM/2) for discretely binned magnitudes. The application uses the half-bin correction explicitly; users should not interpret this as an unbinned estimator.

**Weichert (1980):** recurrence estimation with magnitude-dependent observation durations. The solver is iterative and reports whether the numerical iteration converged; it never silently falls back to a log-linear least-squares fit.

**Stepp (1971/1972):** completeness is examined from cumulative event counts in increasing retrospective observation windows. The plotted standard-deviation statistic is σ(T)=sqrt(λ(T)/T), λ(T)=N(T)/T, with the Poisson −1/2 reference slope.

**Shi & Bolt (1982):** b-value uncertainty is reported with the implemented catalogue-size-based approximation; this should be independently benchmarked against the exact HMTK implementation before regulatory/publication use.

**Gardner & Knopoff (1974):** HMTK-style Type-1 windows. The application currently uses the published/HMTK distance and time fits encoded in the UI and records the method explicitly.

## Important validation status

This repository version is a research prototype. It is not claimed to be numerically bit-for-bit identical to the Python HMTK package. Before using results in a hazard model, benchmark every statistical routine against the installed HMTK/OpenQuake test suite and a published worked example. The application reports Weichert convergence status rather than hiding failure.

## References

Gutenberg, B., & Richter, C. F. (1944). Frequency of earthquakes in California. *Bulletin of the Seismological Society of America, 34*(4), 185–188.

Aki, K. (1965). Maximum likelihood estimate of b in the formula log N = a − bM and its confidence limits. *Bulletin of the Earthquake Research Institute, 43*, 237–239.

Utsu, T. (1965). A method for determining the value of b in a formula log n = a − bM showing the magnitude-frequency relation for earthquakes. *Geophysical Bulletin of Hokkaido University, 13*, 99–103.

Weichert, D. H. (1980). Estimation of the earthquake recurrence parameters for unequal observation periods for different magnitudes. *Bulletin of the Seismological Society of America, 70*(4), 1337–1346.

Stepp, J. C. (1971/1972). Analysis of completeness of the earthquake sample in the Puget Sound area and its effect on statistical estimates of earthquake hazard. *Proceedings of the International Conference on Microzonation*, Seattle, 897–909.

Shi, Y., & Bolt, B. A. (1982). The standard error of the magnitude-frequency b value. *Bulletin of the Seismological Society of America, 72*(5), 1677–1687.

Gardner, J. K., & Knopoff, L. (1974). Is the sequence of earthquakes in Southern California, with aftershocks removed, Poissonian? *Bulletin of the Seismological Society of America, 64*(5), 1363–1367.
