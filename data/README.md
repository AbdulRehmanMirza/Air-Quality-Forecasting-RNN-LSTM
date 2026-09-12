# Dataset

This project uses the **UCI Air Quality Dataset**.

- **Official source:** https://archive.ics.uci.edu/dataset/360/air+quality
- **File used:** `AirQualityUCI.xlsx`
- **Size:** 9,357 hourly observations (March 2004 – April 2005)
- **Missing values:** encoded as `-200` (per the UCI documentation) — handled in the notebook by replacing `-200` with `NaN` and forward/backward-filling.

## How to get the data

1. Go to https://archive.ics.uci.edu/dataset/360/air+quality
2. Download the dataset (it ships as a zip containing `AirQualityUCI.xlsx` and a CSV variant)
3. Place `AirQualityUCI.xlsx` in this `data/` folder, so the path matches:

   ```
   data/AirQualityUCI.xlsx
   ```

4. Run the notebook — it reads the file from `DATA_PATH = "data/AirQualityUCI.xlsx"`.

The dataset itself is not committed to this repository (to keep the repo lightweight and respect the original distribution terms).

## Citation

> Vito, S. (2008). *Air Quality*. UCI Machine Learning Repository. DOI: 10.24432/C59K5F
