# CelebA — DPS Deblurring + KLIP Anomaly Detection

This directory applies the KLIP method to **image deblurring** on CelebA-HQ 256×256 faces. A pre-trained DDPM (`google/ddpm-celebahq-256`) is used as the diffusion prior. DPS (Diffusion Posterior Sampling) runs the reverse diffusion while storing measurement updates at every step, which are then used to build a pixel-level anomaly heatmap.



## Setup

### 1. Create a Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate       # Linux / macOS
# .venv\Scripts\activate        # Windows
```

### 2. Install PyTorch

Install PyTorch first with the correct CUDA wheel for your system (example below uses CUDA 12.1):

```bash
pip install torch==2.3.1 torchvision==0.18.1 torchaudio==2.3.1 \
    --index-url https://download.pytorch.org/whl/cu121
```

### 3. Install Remaining Dependencies

```bash
pip install \
  numpy==1.26.4 \
  scipy==1.12.0 \
  Pillow==10.4.0 \
  matplotlib==3.8.4 \
  scikit-learn==1.4.2 \
  tqdm==4.66.5 \
  diffusers==0.30.3 \
  transformers==4.44.2 \
  accelerate==0.33.0 \
  huggingface-hub==0.24.6 \
  safetensors==0.4.5 \
  jupyterlab==4.2.5 \
  ipykernel==6.29.5
```

### 4. Prepare Your Data

The notebook expects face images as `.png` files and (for AUC evaluation) paired binary masks as `<stem>_mask.png`. Organise them like this:

```
test_artifact_added/
├── 00002.png
├── 00002_mask.png
├── 00015.png
├── 00015_mask.png
└── ...
```

To generate synthetic artifact images and masks from clean CelebA faces, use the **Section 11** cells in the notebook (no extra data needed beyond a clean face image and a transparent-background artifact PNG).

---

## Usage

### Launch the Notebook

```bash
jupyter lab
```

Open `celebA.ipynb`. At the very top of the notebook there is a **Configuration** cell — set all your paths and parameters there before running anything else:

```python
# ── SET YOUR PATHS HERE ───────────────────────────────────────────────
INPUT_IMAGE_PATH = "image copy 44"      # path stem (no .png) for single-image runs
TEST_IMAGE_DIR   = "./test_artifact_added"  # directory with image + mask pairs
ARTIFACT_OUT_DIR = "./test_artifact_added"  # where synthetic artifacts are saved
ARTIFACT_IMAGE_PATH = "./pngegg.png"    # transparent-background artifact PNG
DEVICE_STR       = "cuda:0"             # GPU device ("cpu" if no GPU)
# ─────────────────────────────────────────────────────────────────────

# ── Diffusion / DPS parameters ────────────────────────────────────────
NUM_SAMPLES      = 8      # parallel noisy samples per image
NUM_INFERENCE_STEPS = 100
DPS_SCALE        = 0.3
SIGMA_Y          = 1.0

# ── Heatmap window ────────────────────────────────────────────────────
START_WINDOW     = 65     # first diffusion step index to accumulate
STOP_WINDOW      = 85     # last  diffusion step index (exclusive)
```

### Notebook Sections

| Section | What it does |
|---|---|
| 1 — Imports & reproducibility | Sets seeds, imports all libraries |
| 2 — Load pretrained DDPM | Downloads `google/ddpm-celebahq-256` from Hugging Face |
| 3 — Blur operator & image utils | Defines Gaussian blur `H`, `H^T`, PIL↔tensor helpers |
| 4 — DPS core | Single measurement-update step function |
| 5 — DPS sampling (stores updates) | Full reverse diffusion; **measurement updates stored here** |
| 6 — Run DPS on a single image | Loads image, builds blurry measurement, runs DPS |
| 7 — Anomaly heatmap | Aggregates stored updates → `(256,256)` heatmap |
| 8 — Visualisation | Heatmap overlay on original image |
| 9 — AUC evaluation | Batch AUC over a dataset with masks |
| 10 — Streaming heatmap | Memory-efficient on-the-fly heatmap (no update storage) |
| 11 — Synthetic artifact generation | Composites scar overlay onto faces; saves binary masks |
| 12 — Runtime benchmark | Compares sampling time with and without heatmap |

---

## How the Anomaly Heatmap Works

At each reverse diffusion step the DPS correction `Δx_t = x_t^{after DPS} − x_t^{before DPS}` is saved. After sampling is complete, the heatmap is computed as:

```
heatmap(i,j) = Σ_{t=START_WINDOW}^{STOP_WINDOW}  (Δx_t(i,j) / √β_t)²
```

summed over the batch and colour channels. Pixels where the measurement update is consistently large within the chosen timestep window correspond to regions the model cannot explain with the prior — flagging them as anomalous.

---

## Dependencies

| Package | Version |
|---|---|
| numpy | 1.26.4 |
| scipy | 1.12.0 |
| Pillow | 10.4.0 |
| matplotlib | 3.8.4 |
| scikit-learn | 1.4.2 |
| tqdm | 4.66.5 |
| torch | 2.3.1 |
| torchvision | 0.18.1 |
| torchaudio | 2.3.1 |
| diffusers | 0.30.3 |
| transformers | 4.44.2 |
| accelerate | 0.33.0 |
| huggingface-hub | 0.24.6 |
| safetensors | 0.4.5 |
| jupyterlab | 4.2.5 |
| ipykernel | 6.29.5 |
