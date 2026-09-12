# Air Quality Forecasting with SimpleRNN and LSTM

[![Python](https://img.shields.io/badge/Python-3.10-blue.svg)](https://www.python.org/)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange.svg)](https://www.tensorflow.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Forecasting Carbon Monoxide (CO) concentration **6 hours into the future** from **72 hours** of historical air-quality sensor data, comparing a vanilla **SimpleRNN** against an **LSTM** on an identical pipeline.

## Table of Contents

- [Highlights](#highlights)
- [Problem Statement](#problem-statement)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Models](#models)
- [Results](#results)
- [Key Findings](#key-findings)
- [Known Issues Worth Understanding](#known-issues-worth-understanding)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Technologies](#technologies)
- [Future Improvements](#future-improvements)

## Highlights

- **Task:** univariate time-series forecasting — predict CO(GT) 6 hours ahead from a 72-hour lookback window
- **Models compared:** SimpleRNN vs. LSTM, trained under identical conditions (same architecture depth, optimizer, learning rate, batch size, epochs) so the comparison isolates the effect of the recurrent layer
- **Result:** LSTM reduced test MSE by ~10.7% over SimpleRNN, consistent with the vanishing-gradient behavior expected from a vanilla RNN over a 72-step window
- **Reproducibility:** fixed random seeds, no shuffling of time-ordered data, chronological train/val/test split performed *before* scaling to avoid leakage (see [Known Issues](#known-issues-worth-understanding))

## Problem Statement

The goal is to forecast **CO concentration 6 hours ahead**, using the previous **72 hours** of hourly air-quality observations. This is a deliberately non-trivial horizon: the model can't just repeat the last observed value — it has to pick up on longer-range temporal structure (like daily rush-hour pollution cycles) to do well.

## Dataset

| | |
|---|---|
| **Name** | UCI Air Quality Dataset |
| **Source** | [archive.ics.uci.edu/dataset/360/air+quality](https://archive.ics.uci.edu/dataset/360/air+quality) |
| **Observations** | 9,357 hourly readings (March 2004 – April 2005) |
| **Origin** | Gas multisensor device deployed in an Italian city |
| **Target variable** | `CO(GT)` — ground-truth CO concentration (mg/m³) |
| **Other features (unused here)** | NMHC, C6H6, NOx, NO2, O3-related sensor responses, temperature, relative humidity, absolute humidity |
| **Missing values** | Encoded as `-200` (per UCI documentation) |
| **Time-series nature** | Strictly chronological hourly data; never shuffled |

This project uses **only** the target's own history (univariate forecasting). See [Future Improvements](#future-improvements) for a multivariate extension.

> The dataset file (`AirQualityUCI.xlsx`) is not included in this repository. See [`data/README.md`](data/README.md) for download instructions and citation.

## Methodology

1. **Preprocessing** — replace `-200` sentinels with `NaN`, then forward-fill / back-fill to keep the hourly series continuous.
2. **Train / validation / test split** — chronological 80% / 10% / 10% split, performed *before* scaling.
3. **Normalization** — `MinMaxScaler` fit **only on the training split**, then applied to validation/test data.
4. **Sequence creation** — sliding windows of the past 72 hours (`LOOKBACK = 72`) used to predict the CO value 6 hours later (`AHEAD = 6`).

```
[ t-72 ... t-1 ]  ──►  model  ──►  ŷ = CO at time (t + 6)
   72 hourly readings                  6-hour-ahead forecast
```

## Models

Both models share the same surrounding architecture — a single 64-unit recurrent layer → 32-unit dense layer (ReLU) → 1-unit output — trained with Adam (lr = 0.001), batch size 64, for 30 epochs.

### SimpleRNN
A baseline vanilla recurrent layer. Vanilla RNNs are trained via Backpropagation Through Time, which becomes unstable over long sequences — with a 72-step lookback, gradients from early timesteps tend to vanish, limiting how much of the window the model can actually use.

### LSTM
The same architecture with `SimpleRNN` swapped for `LSTM`. LSTM's gated cell state (forget / input / output gates) lets information flow across many timesteps without the same gradient decay, making it much better suited to capturing dependencies across a 72-hour window.

## Results

> ⚠️ These are the results from the original experiment run, produced before the scaling fix described in [Known Issues](#known-issues-worth-understanding). Re-run the notebook end-to-end with the corrected preprocessing and update this table before treating these as final.

| Model     | MAE    | RMSE   | R²     |
|-----------|--------|--------|--------|
| SimpleRNN | 0.6784 | 0.9163 | 0.5293 |
| LSTM      | **0.6296** | **0.8658** | **0.5797** |

LSTM reduced test MSE by **~10.7%** relative to SimpleRNN in this run.

## Key Findings

**LSTM outperformed SimpleRNN on every metric** — lower error (MAE, RMSE) and higher explained variance (R²). This is consistent with the theory: forecasting 6 hours ahead from a 72-hour window requires the model to retain information across many timesteps, which is exactly where SimpleRNN's vanishing-gradient limitation shows up, and exactly what LSTM's gating mechanism is designed to address.

## Known Issues Worth Understanding

While putting this project together, one methodological issue was identified and fixed:

- **Scaling leakage** — the original version fit `MinMaxScaler` on the *entire* CO series (train + validation + test) before splitting. Because the scaler's min/max statistics were computed with knowledge of future (validation/test-period) values, this is a form of data leakage. The corrected notebook fits the scaler **only on the training split** and applies it to the rest. Since CO concentration doesn't have wildly different min/max values in the held-out period, the practical effect on the reported metrics is expected to be small — but the notebook should be re-run to confirm before the results above are treated as final.

## Project Structure

```
air-quality-forecasting-rnn-lstm/
│
├── README.md
├── air_quality_forecasting_rnn_lstm.ipynb
├── requirements.txt
├── data/
│   └── README.md
└── images/
```

## How to Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/air-quality-forecasting-rnn-lstm.git
   cd air-quality-forecasting-rnn-lstm
   ```
2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```
3. **Obtain the dataset** — see [`data/README.md`](data/README.md) for the download link and citation.
4. **Place the dataset** at `data/AirQualityUCI.xlsx`.
5. **Run the notebook**
   ```bash
   jupyter notebook air_quality_forecasting_rnn_lstm.ipynb
   ```
   Run all cells top to bottom to regenerate training curves, evaluation metrics, and comparison plots with the corrected, leakage-free preprocessing.

## Technologies

Python · NumPy · Pandas · Matplotlib · Scikit-learn · TensorFlow / Keras · Jupyter Notebook / Google Colab

## Future Improvements

- Compare against a **GRU** model
- Systematic **hyperparameter tuning**
- Experiment with different **lookback windows** and **forecast horizons**
- **Multi-step forecasting** instead of a single point 6 hours ahead
- **Multivariate** input using the other pollutant/weather sensor columns
- **Attention-based** architectures
- Deploy the trained model as an **API** for real-time air-quality forecasting
