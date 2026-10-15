# Neuro-DT Dashboard

A Streamlit-based clinical interface for the Brain Digital Twin (BDT): a hybrid 3D CNN-Transformer Alzheimer's Disease classifier paired with an empirically estimated, APOE4-stratified Markov Chain progression model. The dashboard wraps a trained checkpoint, the fitted transition matrices, and live SHAP/Grad-CAM explainability behind one interface, so a diagnosis, its explanation, and a what-if intervention simulation are available from a single screen rather than from separate research scripts.

This repository accompanies two papers:

- **Brain Digital Twin (BDT)** — the full research paper describing the classifier, training, and Markov simulation engine this dashboard serves. *(citation / link to be added once published)*
- **Neuro-DT Dashboard** — the SoftwareX companion paper describing this software specifically. *(citation / link to be added once published)*

## What it does

| Panel | Description |
|---|---|
| Prediction | Runs the classifier on a selected patient; shows the predicted diagnosis, confidence, and per-class probability |
| About / model information | Reports architecture, parameter count, and per-class validation AUC for the active checkpoint |
| 5-year state trajectories | Monte Carlo simulation across four scenarios (baseline, APOE4-positive untreated, +Lecanemab, +Donanemab) with 95% confidence intervals |
| Dementia risk factor | Single-chart comparison of 5-year dementia probability across those four scenarios |
| Grad-CAM explainability | Live-generated axial/sagittal/coronal activation overlay for the selected patient |
| Lifestyle recommendations | Structured, AI-generated lifestyle guidance plus downloadable patient/clinician report exports |

## Repository contents

```
neuro-dt-dashboard/
├── app.py                 # Streamlit application — all dashboard panels
├── model.py                # MultimodalTransformer architecture (3D DenseNet-121 + Transformer)
├── strip_checkpoint.py      # Utility: shrinks a full training checkpoint to an inference-only one
├── Dockerfile                # Container build used for the Azure App Service deployment
├── requirements.txt            # Python dependencies
├── screenshots/                  # Panel screenshots, also used in the SoftwareX paper
├── LICENSE
└── README.md
```

Trained model checkpoints and the ADNI tensor cache are **not** included in this repository. They are loaded at runtime from `CHECKPOINT_DIR` / `CACHE_DIR`, or fetched from Azure Blob Storage if a scan isn't already cached locally. ADNI data is restricted-access; see the Data availability note below.

## Setup

```bash
pip install -r requirements.txt
```

Point `CHECKPOINT_DIR` at a directory containing a stripped checkpoint (see `strip_checkpoint.py`) before running. On CPU-only machines, install the CPU builds of PyTorch/TorchVision exactly as pinned in `requirements.txt` and `Dockerfile`.

## Configuration (environment variables)

None of the following are committed to this repository. Set them in your shell, a local `.env` file (already gitignored), or your deployment platform's application-settings/secrets store — never hardcode them in source.

| Variable | Required | Purpose |
|---|---|---|
| `CHECKPOINT_DIR` | No (defaults to `./checkpoints`) | Local directory holding the model checkpoint |
| `CACHE_DIR` | No (defaults to `./tensor_cache`) | Local directory for cached preprocessed MRI tensors |
| `GOOGLE_API_KEY` | No | Enables live LLM-generated lifestyle recommendations (Gemini); without it, the app falls back to static rule-based recommendations |
| `AZURE_TENANT_ID` | Only if fetching scans from Azure Blob Storage | Azure AD tenant ID for the app registration used to read the ADNI blob container |
| `AZURE_CLIENT_ID` | Only if fetching scans from Azure Blob Storage | Azure AD application (client) ID |
| `AZURE_CLIENT_SECRET` | Only if fetching scans from Azure Blob Storage | Azure AD client secret — **never commit this** |
| `AZURE_STORAGE_ACCOUNT_URL` | Only if fetching scans from Azure Blob Storage | e.g. `https://<account>.blob.core.windows.net` |
| `AZURE_STORAGE_CONTAINER` | No (defaults to `adni-data`) | Blob container name holding the DICOM archives |

Without the Azure variables set, the dashboard still runs and performs tabular-only inference; it just can't fetch a scan that isn't already in the local tensor cache.

## Running locally

```bash
export CHECKPOINT_DIR=./checkpoints
streamlit run app.py
```

## Running with Docker

```bash
docker build -t neuro-dt-dashboard .
docker run -p 8000:8000 \
  -e CHECKPOINT_DIR=/app/checkpoints \
  -v $(pwd)/checkpoints:/app/checkpoints \
  neuro-dt-dashboard
```

The app listens on port 8000 inside the container, matching the Azure App Service deployment this Dockerfile was built for.

## Data availability

The imaging and clinical data used to train the underlying classifier come from the Alzheimer's Disease Neuroimaging Initiative (ADNI, [adni.loni.usc.edu](https://adni.loni.usc.edu)). ADNI data is available to qualified researchers under ADNI's standard data-use agreement and is not redistributed here.

## License

MIT — see [LICENSE](LICENSE).
