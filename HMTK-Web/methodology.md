# HMTK-Web Methodology Appendix

## 1. Catalogue
The importer mirrors the common HMTK field names: eventID, Agency, year, month, day, hour, minute, second, longitude, latitude, depth, depthError, mag, magType and sigmaMag. All computations operate on the loaded catalogue in the browser.

## 2. Gardner–Knopoff
The current build exposes Gardner–Knopoff Type 1 as an optional preprocessing step. It processes larger events first and flags smaller events inside the magnitude-dependent space/time window. It does not silently implement AFTERAN/Reasenberg.

## 3. Stepp
For each magnitude bin and retrospective observation window T, N(T) is counted from the catalogue end, λ(T)=N(T)/T and σ(T)=sqrt(λ(T)/T). The −1/2 log-log slope is the stationary Poisson reference. The application gives a suggested departure year but preserves a user-editable final year.

## 4. Aki/Utsu
For a single complete period, the implemented discrete-magnitude estimator is b=log10(e)/(Mbar−Mmin+ΔM/2). The annual rate is N/Δt and a=log10(rate)+bMmin.

## 5. Weichert
The unequal-duration method uses the magnitude-bin observation durations implied by the completeness table and iteratively updates b. The browser reports convergence status and iteration failure; it does not substitute ordinary least squares.

## 6. Unimplemented/limited claims
The current build does not claim a complete reproduction of every historical HMTK class, every plotting interaction, every exact Shi–Bolt implementation detail, or a validated reproduction of a published Weichert numerical table. These are explicit validation targets for the next release.
