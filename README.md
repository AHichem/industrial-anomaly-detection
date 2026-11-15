# Industrial anomaly detection on a fermentation process

Detecting process faults in 100 batches of a simulated 100,000 L penicillin fermentation (IndPenSim), from simple statistical baselines to supervised and unsupervised machine learning.

## What this project shows

- **Data quality:** finding and fixing labelling errors in a raw industrial dataset before any modelling.
- **Exploratory analysis** of multivariate time series (batch trajectories, control strategies, fault timing).
- **Baselines first:** a golden batch envelope, used to measure what machine learning actually adds.
- **Machine learning:** unsupervised (Isolation Forest, K-means, PCA, autoencoder) and supervised (Random Forest) anomaly detection.
- **Sound evaluation:** train/test split by batch to avoid data leakage, false alarm rate vs detection rate.

## The problem

In biopharmaceutical manufacturing, every batch is expensive and quality must be demonstrated, not assumed. A monitoring model should flag a batch that is drifting before it is lost. Two errors matter: a **false alarm** stops a good batch and erodes the operators' trust, a **missed fault** loses the batch. A useful monitor keeps both low.

## Data

[IndPenSim](https://data.mendeley.com/datasets/pdnjz7zz5x) (Goldrick et al.), licence CC BY 4.0. 100 fed-batch runs, about 1,100 time points each and 23 online process variables:

| Batches | Control strategy |
|---|---|
| 1-30 | Fixed recipe |
| 31-60 | Operators |
| 61-90 | Advanced process control (APC) using Raman spectroscopy |
| 91-100 | Contain process faults |

The data (about 2.5 GB) is not stored in this repository. Notebook 01 downloads it from Mendeley and caches a compact copy of the process variables in Google Drive.

## Notebooks

| Notebook | Content | Methods | Status |
|---|---|---|---|
| 01 | Load IndPenSim, sanity checks, first look at the 100 batches | Data validation, exploratory analysis | Done |
| 02 | Golden batch (global and per control strategy); golden batch vs a first ML detector on false alarms; can the 3 control strategies be found without labels? | Mean ± 3σ envelope, **Isolation Forest**, **K-means** | Done |
| 03 | Supervised fault detection: features computed on time windows, model trained on labelled faults | Feature engineering, **Random Forest**, precision / recall, feature importance | In progress |
| 04 | Unsupervised detection trained on normal batches only, compared with all previous models | **PCA** (T² and SPE), **autoencoder** | Planned |
| 05 (optional) | Soft sensor: predict penicillin from the online sensors | Regression | Optional |

## Key findings so far

**Data (notebook 01)**
- The raw file contains three labelling errors (swapped columns, a `Batch ID` and a `Fault flag` that are neither); the batch identifier is validated against the time resets at every load.
- Batch duration and penicillin alone are not fault indicators: faulty batches stay within the normal duration range, and many fault-free batches also show a penicillin drop after ~150 h.
- Faults are intermittent and start part-way through the batch, which makes detection delay a key metric.

**Baselines (notebook 02)**
- The golden batch envelope is wide, and faulty batches can stay inside it. Splitting it by control strategy barely helps: the three strategies follow almost the same mean trajectory.
- Used as a monitor (one variable at a time), the golden batch raises a false alarm on all 30 normal test batches (15.5 % of the points).
- A first Isolation Forest cuts false alarms to 1 %, but barely detects the faults (2 % of points in alarm on faulty batches): it judges each point alone and ignores time.
- K-means with k = 3 does not recover the three control strategies from batch-level summaries.

## How to run

Open the notebooks in Google Colab and run notebook 01 once: it downloads the data and caches it in Google Drive. The other notebooks read that cached copy.

## Next steps

- Robustness: inject sensor faults (bias, drift, frozen values, missing data) and measure how each model degrades.
- Gradient boosting (XGBoost) as a second supervised model.
- A small dashboard to show alarms to a non-technical user.

## References

- Goldrick S., Stefan A., Lovett D., Montague G., Lennox B. (2015). The development of an industrial-scale fed-batch fermentation simulation. *Journal of Biotechnology*, 193, 70-82.
- Goldrick S., Duran-Villalobos C., Jankauskas K., Lovett D., Farid S. S., Lennox B. (2019). Modern day monitoring and control challenges outlined on an industrial-scale benchmark fermentation process. *Computers and Chemical Engineering*, 130, 106471.
