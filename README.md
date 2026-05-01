# Magnetic-Complexity-Vs-UV-emission
UV intensity time series of solar active regions using Aditya-L1 SUIT NB03 (Mg II k, 2796 Å) + SDO/HMI SHARP magnetic masks. Pixel-precise AR photometry with quiet-Sun normalization. Python · SunPy · JSOC · DRMS

# UV Analysis from Active Regions — SUIT NB03 + HMI SHARP

**Aditya-L1 SUIT × SDO/HMI** | UV intensity time series of sunspot active regions using exact SHARP magnetic masks

***

## What This Does

This notebook extracts ultraviolet (UV) intensity time series from solar active regions (ARs) observed by the **SUIT NB03** filter (Mg II k, 2796 Å) aboard India's **Aditya-L1** spacecraft, cross-referenced with **HMI SHARP** magnetic field data from NASA's **SDO**.

Instead of using a rough bounding box, it **reprojects the exact SHARP bitmap mask** onto each SUIT frame — so the photometric extraction is pixel-precise to the actual magnetically defined AR boundary.

### Pipeline Overview

```
SUIT NB03 FITS files (full-disk UV images)
         │
         ▼
Match each frame to nearest HMI SHARP record (≤20 min)
         │
         ├──▶ Download HMI SHARP magnetogram + bitmap for each target AR
         ├──▶ Download HMI full-disk magnetogram (for quiet-Sun reference)
         │
         ▼
Reproject SHARP bitmap onto SUIT WCS frame
         │
         ▼
Compute AR mask (bitmap ≥ 33/34) and quiet-Sun mask (|B| < 20 G at disk center)
         │
         ▼
Extract mean & median intensity (DN) per AR per frame
         │
         ▼
Save time-series CSV + plots
```

***

## Observations Covered

| Date | Active Regions | Notes |
|------|---------------|-------|
| 4 February 2026 | AR14366 (βγδ), AR14362 (βγ), AR14367 (β), AR14369 (α) | AR14366 was the most complex — Beta-Gamma-Delta class with 1100 µH area |

The Solar Region Summary (NOAA SWPC SRS #35, issued 0030Z 4 Feb 2026) is embedded in the notebook for traceability.

***

## Instruments & Data Sources

| Instrument | Mission | Data Product | What It Provides |
|------------|---------|-------------|-----------------|
| **SUIT NB03** | Aditya-L1 (ISRO) | Full-disk FITS images, Mg II k (2796 Å) | UV chromospheric intensity |
| **HMI SHARP** | SDO (NASA) | `hmi.sharp_720s` magnetogram + bitmap | AR magnetic field + boundary masks |
| **HMI Full-Disk** | SDO (NASA) | `hmi.M_720s` magnetogram | Quiet-Sun LOS magnetic field |
| **NOAA SRS** | SWPC | Daily Solar Region Summary | NOAA AR numbers, classifications |

SHARP data is accessed via **JSOC** (Joint Science Operations Center) using the `drms` Python client.

***

## Repository Structure

```
UV_analysis_from_sunspots.ipynb    ← Main analysis notebook
README.md                          ← You are here
requirements.txt                   ← Python dependencies
.gitignore                         ← Ignores FITS files, CSVs, and large outputs
```

***

## Setup & Usage

### Prerequisites

- Python 3.9+
- A JSOC-registered email address (register free at [http://jsoc.stanford.edu/ajax/register_email.html](http://jsoc.stanford.edu/ajax/register_email.html))
- SUIT NB03 FITS files for your target date (from ISRO's data archive)

### Install Dependencies

```bash
pip install -r requirements.txt
```

Or in a Colab cell:

```python
!pip install drms sunpy astropy numpy pandas matplotlib
```

### Configure the Notebook

Open the notebook and update the **USER SETTINGS** block near the top:

```python
DATE_STR      = "2026-02-04"                    # Observation date
TARGET_NOAA   = [14366, 14362, 14367, 14369]    # NOAA AR numbers to analyze
JSOC_EMAIL    = "your.email@example.com"         # Your registered JSOC email
SUIT_GLOB     = "/path/to/*NB03.fits"            # Path to your SUIT files
OUTDIR        = Path("./output")                 # Where to save results
```

### Run

Execute all cells top-to-bottom. The notebook will:

1. Load your SUIT FITS files and parse observation times
2. Query JSOC to resolve NOAA AR numbers → HARPNUM identifiers
3. Download matching SHARP magnetograms and bitmaps on demand
4. Reproject magnetic masks onto SUIT frames
5. Build a time-series CSV with mean/median DN per AR per frame
6. Generate diagnostic plots

***

## Outputs

| File | Description |
|------|-------------|
| `sharp_suit_YYYY-MM-DD/suit_nb03_sharp_timeseries_YYYY-MM-DD.csv` | Main time-series table (time, QS DN, AR DN per region) |
| `suit_nb03_sharp_timeseries_image3_style.png` | Dual-panel mean + median DN time series plot |
| `intensity_vs_total_magnetic_parameters_*.png` | Scatter plots: UV intensity vs. magnetic flux, energy, helicity, free energy |
| `{AR}_{parameter}_scatter.png` | Individual per-AR scatter panels |

### CSV Column Reference

| Column | Description |
|--------|-------------|
| `time` | UTC observation time of SUIT frame |
| `suit_file` | Source FITS filename |
| `QS_mean_DN` | Mean quiet-Sun intensity at disk centre (|B| < 20 G) |
| `QS_median_DN` | Median quiet-Sun intensity |
| `AR{n}_mean_DN` | Mean UV intensity over the AR (SHARP bitmap mask) |
| `AR{n}_median_DN` | Median UV intensity over the AR |
| `AR{n}_npix` | Number of valid pixels in the AR mask |
| `AR{n}_over_QS` | AR mean DN normalized by quiet-Sun mean DN |
| `HMI_ref_trec` | SHARP T_REC used to fetch the matching HMI data |

***

## Key Technical Details

### Why SHARP Bitmaps?

SHARP bitmaps define the "smooth bounding curve" around an active region in HMI patch coordinates. Pixel values ≥ 33 correspond to the area inside the smooth magnetic boundary, giving a physically motivated mask that tracks the AR as it evolves — much better than a fixed rectangular cutout.

### Quiet-Sun Reference

A 100×100 pixel box centred at the heliographic origin (0, 0 arcsec) is used as a quiet-Sun reference. Only pixels where the reprojected HMI full-disk field strength |B| < 20 G are accepted, ensuring the reference sample is genuinely field-free.

### Temporal Matching

Each SUIT frame is matched to the nearest available SHARP record. If the closest record is more than **20 minutes** away, that frame is skipped to avoid comparing data from different solar rotation states.

### Coordinate Reprojection

All HMI maps (both SHARP patches and the full-disk magnetogram) are reprojected onto the SUIT WCS using `sunpy.map.Map.reproject_to()`, which handles the difference in plate scale, projection, and observer position between SDO (L1) and Aditya-L1.

***

## Science Context

The Mg II k line at 2796 Å is a chromospheric diagnostic sensitive to both the temperature structure and the presence of magnetic fields. Active regions appear bright in this band due to enhanced chromospheric heating associated with their magnetic topology.

By comparing the UV brightness of regions with different magnetic classifications (α through βγδ), this analysis explores whether:
- More complex magnetic configurations (higher flare probability) produce systematically higher UV emission
- UV intensity tracks parameters like total magnetic flux, free energy, and current helicity across a day

This is directly relevant to **solar flare precursor studies** and to understanding how SUIT data can complement traditional magnetogram-based flare prediction.

***

## Dependencies

See `requirements.txt` for pinned versions. Core libraries:

- [`drms`](https://docs.sunpy.org/projects/drms/) — JSOC data query and export client
- [`sunpy`](https://sunpy.org/) — Solar physics image handling and WCS reprojection
- [`astropy`](https://www.astropy.org/) — FITS I/O, coordinates, time handling
- `numpy`, `pandas`, `matplotlib` — Numerics, data tables, plotting

***

## Citation & Acknowledgements

If you use this code or methodology in your research, please acknowledge:

- **Aditya-L1 / SUIT** — Indian Space Research Organisation (ISRO)
- **SDO / HMI** — NASA's Solar Dynamics Observatory
- **JSOC** — Joint Science Operations Center, Stanford University
- **NOAA SWPC** — Solar Region Summaries

***

## License

This project is released under the [MIT License](LICENSE). You are free to use, modify, and distribute this code with attribution.

***

## Author

**Suvan** — Master's student, Solar Physics & Helioseismology  
MIT World Peace University (MIT-WPU), Pune, India  
*Working with Aditya-L1 SUIT and SDO/HMI data for solar flare prediction research.*

***

*"The Sun is not silent — it leaves magnetic signatures everywhere, and SUIT is listening in ultraviolet."*
