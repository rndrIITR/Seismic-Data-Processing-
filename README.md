# Seismic-Data-Processing-

A collection of self-contained browser tools for strong-motion processing and ground-motion selection.

## Applications

- **SeismoWeb** — `index.html`: accelerogram processing, baseline correction, filtering, elastic/inelastic response spectra, intensity measures and exports.
- **GM-Select** — `gm-select.html`: offline ground-motion record filtering, target-spectrum definition, record scaling/selection, visual QA and export. It uses a bundled synthetic/illustrative dataset and user-imported records; it does **not** connect to licensed strong-motion databases.

Open either HTML file directly with a browser or publish the repository through GitHub Pages. No build step or backend is required.


## GM-Match

`GM-Match/index.html` is an offline, client-side response-spectrum matching application. It implements an independent literature-based workflow grounded in Lilhanand & Tseng (1988), Abrahamson (1992), Hancock et al. (2006), and Al Atik & Abrahamson (2010). It uses a Web Worker for the iterative matching calculation, Newmark response-spectrum evaluation, wavelet addition, numerical sensitivity/linearized correction, Method A baseline correction, Method B self-tapered wavelets, convergence history, time-history comparison, QC checks, and matched-record/report export.

**Important methodological disclosure:** GM-Match is not a reproduction of Seismosoft SeismoMatch's proprietary internals and must not be expected to produce bit-for-bit identical numerical results. The alpha-model frequency taper is an explicit independent design choice based on the documented behavior, not a verified Seismosoft formula. The application operates only on user-supplied or bundled synthetic/illustrative records and makes no external database calls.

### Literature basis

- Lilhanand, K. & Tseng, W.S. (1988), wavelet addition concept for spectrum-compatible time histories.
- Abrahamson, N.A. (1992), non-stationary spectral matching wavelet approach.
- Hancock, J. et al. (2006), improved wavelet matching method, with baseline correction of the resulting record.
- Al Atik, L. & Abrahamson, N. (2010), improved non-stationary matching using a self-tapering wavelet and dynamic padding concept.
