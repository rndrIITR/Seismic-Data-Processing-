# SeismoWeb — strong-motion signal processing in the browser

A single self-contained HTML file that does what SeismoSignal does: read an
accelerogram, baseline-correct and filter it, integrate it to velocity and
displacement, build elastic and inelastic response spectra, compute the full
set of ground-motion intensity measures, and export everything.

**No installation, no build step, no internet connection, no dependencies.**
Double-click `SeismoWeb.html` and it runs.

---

## 1. Quick start

1. Open `SeismoWeb.html` in Chrome, Edge, Firefox or Safari.
2. Press **Load demo record** in the header (a synthetic 30 s, 0.35 g record is
   generated and processed immediately), or drop your own file on panel 1.
3. Set Δt, units and the acceleration column; the app fills these in from the
   file header when it can.
4. Tick the baseline and filter boxes, press **Process & analyse**.
5. Read the tabs: Overview, Time histories, Response spectra, Fourier & energy,
   Intensity measures, Validation, Export.

### Verifying the app on your own data

The **Validation** tab is the self-check. Load your *raw* record as the working
data set and your *reference processed* record (from SeismoSignal, PRISM, the
recording agency, …) as the reference, tell it which columns hold acceleration,
velocity and displacement, and press **Compare**. You get peak error, maximum
absolute difference, normalised RMS error and correlation for each of the three
traces, plus overlay and residual plots.

The **Numerical self-test** button (header, or Validation tab section B) runs 21
checks of the numerical kernels against closed-form solutions — no data file
needed.

---

## 2. Supported input formats

| Format | Detection |
|---|---|
| Single column, one acceleration per line | any file with one number per line |
| Two columns `time, acceleration` | Δt inferred from the time column |
| Multi-column (e.g. SeismoSignal exports: `t, a, v, d` × 3 components) | column picker; header names are read |
| PEER NGA `.AT2` / free format (`NPTS=`, `DT=`, N values per line) | `NPTS`/`DT` header keywords |
| CSV / TSV / semicolon-separated | separators normalised automatically |
| Fixed-width scientific columns packed without separators | recovered by a coverage-checked regex scan |

Comment and header lines are skipped. `Δt` is picked up from `DT=`, `Time Step:`
or `sampling rate` header lines, or inferred from a uniform time column.
Units are guessed from header text (`(g)`, `units of g`, `cm/s^2`, `gal`, `m/s²`,
…) and fall back to a magnitude heuristic; the sidebar always shows what the
peak of the selected column works out to in g and m/s², so a wrong guess is
immediately visible.

---

## 3. What it computes

### Baseline correction
* pre-event mean removal over a user-defined window
* **constant** — mean removal
* **acceleration polynomial** (order 0–3) — least-squares polynomial fitted to
  the accelerogram and subtracted (the classic SeismoSignal-style correction)
* **velocity polynomial** (order 0–3) — integrate, fit the polynomial to the
  velocity, differentiate it, subtract the resulting acceleration trend
  (the PEER-style workflow), optionally constrained to v(0) = 0
* the removed trend can be overlaid on the acceleration plot

### Filtering
* Butterworth and Chebyshev type I, low-pass / high-pass / band-pass / band-stop,
  order 1–10
* **IIR** implementation: analog prototype → frequency transform → bilinear
  transform with pre-warping → cascaded second-order sections. The coefficients
  are bit-identical to `scipy.signal.butter(..., output='sos')` (verified to
  < 1e-15 across nine filter designs), and the SOS cascade keeps low-corner
  high-pass filters numerically stable where a direct-form implementation
  would fall apart.
* **Causal** (single forward pass) or **acausal** (forward–backward, zero phase,
  with odd extension padding and steady-state initial conditions — the same
  scheme as `scipy.signal.sosfiltfilt`)
* **FFT** implementation: analytic Butterworth magnitude applied in the
  frequency domain (zero phase by construction)
* live amplitude-response plot in the sidebar
* baseline → filter or filter → baseline, selectable

### Integration
Cumulative trapezoidal rule, `v(0) = d(0) = 0`, applied to acceleration and then
to velocity.

### Response spectra
* **Nigam & Jennings (1969)** exact solution for a piecewise-linear ground
  acceleration (default) or **Newmark-β** with average acceleration
  (β = 1/4, γ = 1/2)
* the accelerogram is linearly re-sampled where necessary so that the
  integration step never exceeds `T / (steps per period)` — without this the
  peak response at short periods is systematically wrong for coarsely sampled
  records. Accuracy setting: 10 / 40 / 80 steps per period.
* several damping ratios at once (enter `2,5,10`)
* outputs `Sd`, `Sv`, `Sa` (absolute), `PSv = ω·Sd`, `PSa = ω²·Sd`, plus an
  ADRS (`Sa` vs `Sd`) plot for capacity-spectrum work
* **inelastic constant-ductility spectra**: bilinear kinematic hardening solved
  by Newmark-β with Newton–Raphson equilibrium iteration, and bisection on the
  yield strength until the target ductility is reached. Reports the yield
  strength coefficient `Cy = Fy/(m·g)`, the inelastic displacement and the
  strength-reduction factor `Rμ`.

### Intensity measures
PGA, PGV, PGD and their times of occurrence · PGV/PGA · a<sub>rms</sub>,
v<sub>rms</sub>, d<sub>rms</sub> · sustained maximum acceleration and velocity
(3rd peak) · effective design acceleration (9 Hz low-pass) · A95 · Arias
intensity I<sub>a</sub> · characteristic intensity I<sub>c</sub> · specific
energy density · CAV, CAV5, EPRI standardised CAV · t<sub>5</sub>, t<sub>75</sub>,
t<sub>95</sub> · significant duration D<sub>5-75</sub> and D<sub>5-95</sub> ·
bracketed and uniform duration · mean period T<sub>m</sub> (Rathje) ·
predominant period of the Fourier spectrum and of S<sub>a</sub> · ASI, VSI,
Housner intensity, EPA, EPV.

### Plots
Custom canvas renderer (nothing is loaded from the network). Every plot supports
drag-box zoom, wheel zoom, shift-drag pan, double-click reset, crosshair
read-out, legend click-to-hide, and PNG / SVG export. Traces are drawn with
min–max pixel-bucket decimation, so a 250 000-point record renders instantly and
still shows every peak.

### Export
Time histories (CSV) · response spectra (CSV) · inelastic spectra (CSV) ·
Fourier/PSD (CSV) · intensity measures (CSV, JSON or clipboard) · everything
(JSON) · a self-contained HTML **summary report** with all metadata, processing
parameters, embedded plots and the full measure table — press *Print → Save as
PDF* in the browser for a PDF.

---

## 4. Files

```
SeismoWeb.html                     the application - this is all you need
seismo_batch.py                    optional command-line batch companion
requirements.txt                   numpy, scipy  (only for seismo_batch.py)
examples/
  example1_peer_nearfault.at2      PEER free format, near-fault velocity pulse
  example2_baseline_drift.csv      two-column CSV with a linear baseline drift
  example3_single_column_gal.txt   single column in gal, with a DC offset
README.md
```

### Optional Python batch companion

`seismo_batch.py` mirrors the same algorithms with numpy/scipy for folders of
records, and doubles as an independent cross-check of the browser results.

```bash
pip install -r requirements.txt

python seismo_batch.py "records/*.at2" --units g \
       --baseline vel-poly --bl-order 1 \
       --filter bandpass --f1 0.1 --f2 25 --order 4 --out results/

# compare a raw record processed here against a reference corrected file
python seismo_batch.py raw.txt --acc-col 1 --dt 0.01 --units g \
       --filter bandpass --f1 1 --f2 25 --order 4 \
       --validate corrected.txt --ref-cols 1,2,3
```

---

## 5. Validation

**Against a real SeismoSignal processing pair.** The record
`2012-01-04-1632-04S_SMLA_E` (24 118 samples, Δt = 0.01 s) was supplied both as
recorded and as processed by SeismoSignal. The processing was first identified
from the spectral ratio of the two files — a **causal 4th-order Butterworth
band-pass, 1–25 Hz, with no baseline correction** — and then reproduced:

| component | quantity | computed | reference | peak error | max abs. difference | RMS error | correlation |
|---|---|---|---|---|---|---|---|
| X | acceleration | 5.78418e-5 g | 5.78500e-5 g | −0.014 % | 1.2e-8 g | 0.055 % | 0.99999986 |
| X | velocity | 2.35462e-3 cm/s | 2.35505e-3 cm/s | −0.018 % | 4.3e-7 cm/s | 0.018 % | 1.00000000 |
| X | displacement | 1.05253e-4 cm | 1.05560e-4 cm | −0.291 % | 4.9e-7 cm | 0.330 % | 0.99999654 |

The reference file is printed to 8 decimals in g, i.e. a quantisation step of
1e-8 g — the acceleration agreement is at that printing floor. The residual on
displacement is the accumulation of that same rounding through two
integrations of a record whose peak displacement is only 1e-4 cm.

**Closed-form checks** (the built-in self-test, 21 assertions):

| kernel | check | agreement |
|---|---|---|
| FFT / DFT | Parseval identity and inverse round trip, radix-2 and Bluestein (N = 4093) | ~1e-13 |
| Butterworth | \|H\| = 1/√2 at every corner, LP / BP, orders 1–8 | < 1e-9 |
| Butterworth | full response vs the analytic bilinear-transform formula | < 1e-15 |
| Integration | double integration of a constant vs the analytic parabola | < 1e-13 |
| SDOF exact | harmonic input vs the closed-form transfer function (Sd, Sv, Sa) | < 3e-6 |
| SDOF Newmark | same | < 3e-6 |
| Baseline | recovery of an injected quadratic drift | < 1e-14 |
| Causal IIR | tone gain vs the designed response, pass-band and stop-band | < 1e-13 |
| Acausal IIR | tone gain = \|H\|² (zero phase) | < 4e-4 |
| FFT filter | tone gain vs the analytic Butterworth magnitude | < 2e-3 |
| Arias, CAV | analytic values for a sinusoid | < 4e-6 |
| Spectra | exact vs Newmark-β over six periods | < 0.1 % |

**Cross-implementation.** The browser engine and the independent
numpy/scipy implementation in `seismo_batch.py` were run on the same record with
identical settings. Every intensity measure agrees to ≤ 6e-12 relative, and the
whole 60-period response spectrum to 4e-9.

**Filter design vs scipy.** The JavaScript `butter`/`cheby1` → zpk → SOS chain
was compared with `scipy.signal` for nine designs (low-pass, high-pass,
band-pass, band-stop, orders 1–8, corners from 0.05 Hz to 40 Hz): worst
coefficient difference 6.7e-16.

**Spectral convergence.** With the default "Standard" accuracy (≥ 40 integration
steps per period, minimum 2× refinement) the acceleration spectrum is within
0.23 % of a fully converged reference over 0.01–5 s, mean error 0.03 %.

---

## 6. Performance

Measured in headless Chromium on the reference container:

| task | time |
|---|---|
| 10 001 points at 200 Hz: baseline + filter + integration + intensity measures + 150-period spectrum + FAS | **83 ms** |
| 6 001 points, full UI analysis and redraw | 323 ms |
| 24 118 points, 100-period spectrum, full UI | ~1 s |
| inelastic constant-ductility spectrum, 40 periods, μ = 4 | ~1.1 s |

The spec's target was "under 5 s for 10 000 points at 200 Hz". Long computations
are chunked so the progress bar keeps moving and the interface stays responsive.

---

## 7. Known limitations

* **Sampling rate is assumed uniform.** Records with a varying time step are not
  re-sampled; the app reads Δt from the first interval and warns if the time
  column is not uniform. Re-sample such records before loading.
* **Instrument response is not deconvolved.** The app processes accelerograms;
  it does not remove a seismometer transfer function.
* **Rotation / component combination** (e.g. RotD50, orientation-independent
  measures) is not implemented — components are processed one at a time.
* **Inelastic spectra** use a bilinear kinematic-hardening model only. Stiffness-
  and strength-degrading models (Clough, Takeda) are not included. Constant-
  ductility bisection stops at ±0.5 % of the target ductility or 40 iterations;
  very short periods with high ductility can be slow to converge.
* **Housner intensity** is reported at the damping you select rather than at
  Housner's original 20 % — enter `20` in the damping box if you need the
  classical definition.
* The **FFT filter** option pads to a power of two before transforming, so it is
  not exactly equivalent to the IIR path near the corners; the IIR path is the
  one to use when reproducing agency processing.
* **Very large records.** Everything runs on the main thread. Records beyond
  roughly 500 000 samples combined with several damping ratios and inelastic
  spectra will take tens of seconds; use `seismo_batch.py` for that.

## 8. Extending it

The whole numerical engine is the `Seismo` object defined in the first `<script>`
block of `SeismoWeb.html` — it has no DOM dependencies, so it can be lifted out
into a `.js` file and used in Node or a Web Worker unchanged. A small automation
API is exposed on `window.SeismoWeb`:

```js
SeismoWeb.loadText(fileText, 'name.txt');   // add a data set
await SeismoWeb.run();                      // process with the current settings
SeismoWeb.analysis                          // {raw, proc, im, spec, inel, fas, cfg}
SeismoWeb.selfTest()                         // {pass, total, tests}
SeismoWeb.core.responseSpectrum(acc, dt, periods, xi, 'exact')
```

Natural extensions: RotD50 / GMRotI component combination, Arias-intensity-based
automatic corner-frequency selection, a Web Worker pool for batch processing,
conditional mean spectra, and code-spectrum overlays (Eurocode 8, IS 1893,
ASCE 7) drawn on the Sa plot.

---

## 9. Deployment

It is one file, so anything that serves static files works:

* **Locally** — double-click the file. It works from `file://` with no server.
* **Shared drive / intranet** — copy the file where people can reach it.
* **GitHub Pages** — commit `SeismoWeb.html` (rename to `index.html`), enable
  Pages on the repository, done.
* **Netlify / Vercel / S3** — drag the folder into the deploy target.
* **Python one-liner** — `python -m http.server 8000` in the folder, then open
  `http://localhost:8000/SeismoWeb.html`.

No API keys, no backend, and no data ever leaves the machine: files are read
with the browser's `FileReader` and everything is computed locally.
