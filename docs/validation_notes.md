# Data Validation Notes

This document records how the accuracy of extracted datasets was assessed, and what limitations
are known. It exists because "we ran the pipeline and got numbers" is not the same claim as
"the numbers are accurate" — this project distinguishes between **internal consistency** (does
our own pipeline agree with itself) and **accuracy** (does it match ground truth), and both are
addressed below.

## LST (Land Surface Temperature)

LST is derived deterministically from a fixed, published USGS calibration formula applied to the
Landsat 8/9 Collection 2 Level 2 thermal band (`ST_B10`):

```
LST (°C) = ST_B10 × 0.00341802 + 149.0 − 273.15
```

This is not a trained/learned estimate — given the same input pixel, it produces the same output
every time. The main source of uncertainty is the underlying Landsat thermal sensor itself, which
USGS has independently validated (~1–2 K RMSE against ground-based validation sites; see USGS
Landsat Collection 2 Level 2 Science Product documentation). LST and the derived UHI intensity
zones are treated as high-confidence measurements on this basis.

## LULC (Land Use / Land Cover)

LULC classification uses Google's **Dynamic World** product — a trained neural network classifier,
not a fixed formula. Its accuracy is therefore a measured error rate, not a guarantee:

- **Published accuracy** (Brown et al., 2022): ~73–75% overall, globally.
- **Our own manual spot-check** (30 randomly sampled points, 15 per city, checked against
  high-resolution satellite basemap imagery):
  - **Kathmandu: 9/15 = 60% agreement**
  - **Hetauda: 3/15 = 20% agreement** (23% if a "maybe/ambiguous" point is counted as half)

Both results are below Dynamic World's published global average, and Hetauda's is substantially
below it. The discrepancy is concentrated in the **Builtup** class specifically — most sampled
points were labeled Builtup (reflecting that ~67% of Hetauda's classified area is Builtup), and a
large share of those turned out on inspection to be hiking trails, forest, farmland, or a seasonal
river rather than genuine built-up land.

**Interpretation**: Dynamic World appears to systematically over-classify Builtup in Hetauda,
likely because its smaller, more heterogeneous rural-urban-riverine landscape is harder to
separate spectrally than Kathmandu's denser, more spectrally uniform urban core.

**Practical implications for this project's results:**
- Kathmandu's LULC-derived statistics (area %, transition matrix) are comparatively more
  trustworthy (60% match, closer to DW's published range).
- Hetauda's Builtup extent and Agriculture→Builtup conversion figures likely **overstate** true
  built-up growth.
- Hetauda's SUHII may be **understated** rather than overstated, if genuinely cooler
  misclassified pixels (vegetation, farmland) are being averaged into the "urban" LST mean.

This is disclosed here rather than hidden, and should be cited explicitly in the report's
limitations section — finding and reporting this through direct validation is a stronger
methodological position than an unverified citation of Dynamic World's generic global accuracy.

### Suggested report language

> "A manual visual spot-check of 30 randomly sampled points (15 per city) against high-resolution
> basemap imagery found 60% agreement for Kathmandu Valley and 20–23% agreement for Hetauda, both
> below Dynamic World's published global average (~73–75%, Brown et al., 2022). The discrepancy is
> most pronounced for Hetauda's Builtup class, suggesting the classifier over-detects built-up
> land in Hetauda's more heterogeneous rural-urban-riverine landscape compared to Kathmandu's
> denser, more spectrally uniform urban core. LULC-derived statistics for Hetauda, particularly
> Builtup extent and associated SUHII estimates, should be interpreted with this limitation in
> mind."

### Raw spot-check data

See `data/raw/LULC_spotcheck_points.csv` (30 points with coordinates, classified label, and
manual Y/N verification) if it needs to be reproduced or extended with more points.

## UHI Intensity Zones

Derived directly from validated LST using a fixed statistical rule (mean ± n×std thresholds per
year, per city) — not a separate learned classification, so it inherits LST's higher confidence
level. Note the important methodological caveat that these zones are **relative** (thresholds are
recomputed per year from that year's own distribution), so zone-area percentages are naturally
stable across years even as the city genuinely warms in absolute terms — see the discussion in
`02_lulc_uhi_analysis/lulc_uhi_analysis.ipynb`, Section 7b.

## Known Data Gaps

- **2015 Dynamic World substitution**: Dynamic World has no pre-monsoon coverage for 2015 in this
  region; the pipeline substitutes 2016 data for "2015" LULC. This is documented in the GEE
  extraction script (`dwEarliestYear: 2016`) and should be disclosed explicitly wherever 2015 LULC
  figures are cited.
- **Cloud/scene-count gaps in 2015 and 2020 rasters**: fewer available Landsat scenes in these
  years (Landsat 9 did not launch until September 2021, so 2015/2020 rely on Landsat 8 alone)
  increases the chance of small no-data gaps in the LST/LULC composites for those years relative
  to 2025.
