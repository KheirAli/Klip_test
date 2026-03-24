# PaDIS + KLIP: KLIP implementation over PaDIS sampling method for inverse problems


This repository is a **modified fork** of [jasonhu4/PaDIS](https://github.com/jasonhu4/PaDIS), a Patch-based Position-Aware Diffusion Inverse Solver for solving inverse imaging problems (CT reconstruction, deblurring, superresolution).

## What's Different From the Original PaDIS

The original PaDIS performs reconstruction by sampling from a patch-based diffusion model. This fork introduces the following changes:

1. **Modified sampling in `inverse_nodist.py`**: The sampler now **stores intermediate measurement updates** (i.e., the data-consistency corrections applied at each diffusion step). These stored updates are the inputs required by the **KLIP (Karhunen–Loève Image Processing)** post-processing method.

2. **KLIP post-processing notebook**: A Jupyter notebook (`klip_analysis.ipynb`) is included that loads the stored measurement updates and runs the KLIP algorithm to further improve reconstruction quality.

---

## Repository Structure

```
.
├── training-runs/          # Checkpoint directory (created by user, not tracked by git)
├── image_dir/              # Input directory: place your test PNG images here
├── results/                # Output directory: reconstructions and stored updates saved here
├── inverse_nodist.py       # Modified PaDIS reconstruction script (stores measurement updates)
├── klip_analysis.ipynb     # Jupyter notebook: run KLIP method on stored updates
├── environment.yml         # Conda environment with all dependencies
└── README.md
```

---

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/KheirAli/Klip_test.git
cd Klip_test
```

### 2. Create the Conda Environment

```bash
conda env create -f environment.yml
conda activate padis
```

> See `environment.yml` for the full list of dependencies. For CT experiments, also install the ODL package (see [`odl_env.yml`](https://github.com/jasonhu4/PaDIS/blob/main/odl_env.yml) in the original repo).

### 3. Prepare Directories

```bash
# Create a directory for your test images (input)
mkdir image_dir

# Create a directory for results (output)
mkdir results

# Create the checkpoint directory
mkdir training-runs
```

Place your test images (`.png`) inside `image_dir/`. The reconstruction script will process all images in this folder.

---

## Usage

### Step 1 — Train or Download a Checkpoint

Train a model (see the original [PaDIS repo](https://github.com/jasonhu4/PaDIS) for training details), or download a pre-trained CT checkpoint from the original authors.

Place the checkpoint (`.pkl` file) inside `training-runs/`.

### Step 2 — Run PaDIS Reconstruction (Modified)

Set your paths at the top of the command. The two key variables are:

| Variable | Description |
|---|---|
| `IMAGE_DIR` | Path to folder containing input PNG test images |
| `OUTDIR` | Path to folder where reconstructions and measurement updates will be saved |

```bash
# ── SET YOUR PATHS HERE ──────────────────────────────────────────────
IMAGE_DIR="./image_dir"         # <-- folder with your input PNG images
OUTDIR="./results"              # <-- folder where outputs will be saved
CHECKPOINT="./training-runs/your-checkpoint/network-snapshot.pkl"
# ─────────────────────────────────────────────────────────────────────

# Example: 20-view CT reconstruction
python3 inverse_nodist.py \
  --network="${CHECKPOINT}" \
  --outdir="${OUTDIR}" \
  --image_dir="${IMAGE_DIR}" \
  --image_size=256 \
  --views=20 \
  --name=ct_parbeam \
  --steps=100 \
  --sigma_min=0.003 \
  --sigma_max=10 \
  --zeta=0.3 \
  --pad=24 \
  --psize=56
```

**What the modified script saves to `OUTDIR`:**
- Reconstructed images (same as original PaDIS)
- **`measurement_updates/`** — a subdirectory containing the stored measurement update tensors at each diffusion step, required for the KLIP method

### Step 3 — Run KLIP Post-processing

After reconstruction is complete, open and run the Jupyter notebook:

```bash
jupyter notebook klip_analysis.ipynb
```

At the top of the notebook, set the same output directory:

```python
# ── SET YOUR PATHS HERE ──────────────────────────────────────────────
OUTDIR = "./results"            # must match the --outdir used in Step 2
KLIP_OUTDIR = "./results/klip" # where KLIP-processed images will be saved
# ─────────────────────────────────────────────────────────────────────
```

The notebook will:
1. Load the stored measurement updates from `OUTDIR/measurement_updates/`
2. Apply the KLIP algorithm
3. Save the KLIP-processed reconstructions to `KLIP_OUTDIR`

---

## Key Parameters (Reference)

| Parameter | Description | Default |
|---|---|---|
| `--network` | Path to trained model checkpoint (`.pkl`) | required |
| `--outdir` | Output directory for results | required |
| `--image_dir` | Directory of input PNG images | required |
| `--image_size` | Image resolution (pixels) | `256` |
| `--views` | Number of CT views (for CT tasks) | `20` |
| `--steps` | Number of diffusion sampling steps | `100` |
| `--sigma_min` | Minimum noise level | `0.003` |
| `--sigma_max` | Maximum noise level | `10` |
| `--zeta` | Data-consistency step size | `0.3` |
| `--pad` | Patch padding size | `24` |
| `--psize` | Patch size | `56` |
| `--name` | Inverse problem type (e.g. `ct_parbeam`, `deblur`, `superres`) | required |

> All other hyperparameters inherit from the original PaDIS — see [jasonhu4/PaDIS](https://github.com/jasonhu4/PaDIS) for full documentation.

---

## Why We Store Measurement Updates

In the original PaDIS, data-consistency corrections are computed and applied at each diffusion timestep but **discarded** afterwards. For the KLIP method to work, these corrections (measurement updates) must be retained across all timesteps for each reconstructed image.

The modification in `inverse_nodist.py` intercepts these updates during the sampling loop and writes them to disk. The KLIP notebook then loads them as a matrix and applies a truncated eigendecomposition to suppress noise components — yielding cleaner reconstructions than the raw PaDIS output alone.

---

## Citation

If you use this code, please cite the original PaDIS paper:

```bibtex
@article{hu2023padis,
  title   = {PaDIS: Patch-based Diffusion Inverse Solver},
  author  = {Hu, Jason and others},
  year    = {2023},
  url     = {https://github.com/jasonhu4/PaDIS}
}
```

---

## Acknowledgements

This repository is built on top of [jasonhu4/PaDIS](https://github.com/jasonhu4/PaDIS). The KLIP post-processing code in `klip_analysis.ipynb` was developed separately and integrated into this fork to enable improved reconstruction via Karhunen–Loève decomposition of the diffusion measurement updates.
