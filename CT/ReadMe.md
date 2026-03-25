# CT — PaDIS Reconstruction + KLIP

This directory contains the **CT inverse problem** experiments. It is a modified version of [jasonhu4/PaDIS](https://github.com/jasonhu4/PaDIS) where the reverse diffusion sampler stores intermediate measurement updates so that the **KLIP** post-processing method can be applied.

---

## Directory Structure

```
CT/
├── inverse_nodist.py           # Modified PaDIS sampler — stores measurement updates
├── inverse_operators.py        # CT forward operator (H) and adjoint (H^T)
├── Klip_PaDIS.ipynb            # KLIP notebook: loads updates, runs KLIP, scores OOD
├── sampling_script.sh          # Bash script to launch CT reconstruction
├── process_AAPM.py             # Preprocessing script for the AAPM CT dataset
├── dps_sampling_test.py        # Standalone DPS sampling test
├── parbeam_updated.py          # Updated parallel-beam CT geometry
├── odlstuff/                   # ODL-based CT geometry utilities
│   ├── parbeam.py              # Parallel-beam projector
│   ├── parbeam_updated.py      # Updated parallel-beam projector
│   ├── fanbeam.py              # Fan-beam projector
│   ├── CBCTDF.py               # Cone-beam CT geometry
│   ├── odl_2d.py               # 2D ODL helpers
│   ├── odl_demo.py             # ODL demonstration script
│   ├── operator2.py            # Additional operator definitions
│   └── README.md               # ODL setup notes
├── training/                   # PaDIS model architecture and training loop
│   ├── networks.py             # Score network definition
│   ├── training_loop.py        # Training loop
│   ├── dataset.py              # Dataset loader
│   ├── loss.py                 # Training loss
│   ├── augment.py              # Data augmentation
│   ├── patch_loss.py           # Patch-based loss
│   ├── pos_embedding.py        # Positional embedding
│   ├── unet.py                 # UNet backbone
│   ├── fp16_util.py            # Mixed precision utilities
│   └── nn.py                   # Neural network helpers
├── torch_utils/                # PyTorch utilities (from original PaDIS)
├── dnnlib/                     # Deep learning utility library (from original PaDIS)
├── odl_env.yml                 # Conda environment (includes ODL)
└── requirements_CelebA.txt     # Pip requirements
```

---

## Setup

### 1. Create the Conda Environment

ODL (Operator Discretization Library) is required for the CT geometry and is best installed via conda:

```bash
conda env create -f odl_env.yml
conda activate odl_env
```

> If you run into ODL installation issues, refer to `odlstuff/README.md` and the simplified environment file at `odlstuff/odl_env_simplified.yml`.

### 2. Prepare Your Data

Download the **AAPM Low-Dose CT Challenge** dataset and preprocess it:

```bash
python process_AAPM.py --data_dir /path/to/aapm/raw --out_dir ./data
```

Place processed PNG images in a directory of your choice — this will be your `IMAGE_DIR`.

### 3. Prepare Directories

```bash
mkdir -p image_dir results training-runs
```

---

## Usage

### Step 1 — Download or Train a Checkpoint

Download a pre-trained PaDIS CT checkpoint from the [original PaDIS repo](https://github.com/jasonhu4/PaDIS) and place the `.pkl` file inside `training-runs/`.

To train from scratch, see the original PaDIS training instructions.

### Step 2 — Run CT Reconstruction

Open `sampling_script.sh` and set the two key path variables at the top:

```bash
# ── SET YOUR PATHS HERE ───────────────────────────────────────────────
IMAGE_DIR="./image_dir"       # folder containing input PNG images
OUTDIR="./results"            # folder where outputs will be saved
CHECKPOINT="./training-runs/your-checkpoint/network-snapshot.pkl"
# ─────────────────────────────────────────────────────────────────────
```

Then run:

```bash
bash sampling_script.sh
```

This calls `inverse_nodist.py`, which saves to `OUTDIR`:
- Reconstructed images
- `measurement_updates/` — per-step measurement update tensors needed for KLIP

You can also run the script directly:

```bash
python inverse_nodist.py \
  --network="./training-runs/your-checkpoint/network-snapshot.pkl" \
  --outdir="./results" \
  --image_dir="./image_dir" \
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

### Step 3 — Run KLIP (`Klip_PaDIS.ipynb`)

```bash
jupyter lab
```

Open `Klip_PaDIS.ipynb` and set the paths at the top of the notebook:

```python
# ── SET YOUR PATHS HERE ───────────────────────────────────────────────
OUTDIR      = "./results"        # must match --outdir used above
KLIP_OUTDIR = "./results/klip"   # where KLIP outputs will be saved
# ─────────────────────────────────────────────────────────────────────
```

The notebook will:
1. Load the stored measurement updates from `OUTDIR/measurement_updates/`
2. Stack them into a matrix across diffusion timesteps
3. Apply the KLIP algorithm (truncated Karhunen–Loève decomposition)
4. Save improved reconstructions and OOD anomaly scores to `KLIP_OUTDIR`

---

## Key Parameters

| Parameter | Description | Default |
|---|---|---|
| `--network` | Path to model checkpoint (`.pkl`) | required |
| `--outdir` | Output directory | required |
| `--image_dir` | Input PNG directory | required |
| `--image_size` | Image resolution (pixels) | `256` |
| `--views` | Number of CT projection views | `20` |
| `--steps` | Diffusion sampling steps | `100` |
| `--sigma_min` | Minimum noise level | `0.003` |
| `--sigma_max` | Maximum noise level | `10` |
| `--zeta` | Data-consistency step size | `0.3` |
| `--pad` | Patch padding | `24` |
| `--psize` | Patch size | `56` |
| `--name` | Problem type (`ct_parbeam`, `deblur`, `superres`) | required |

---

## Why Measurement Updates Are Stored

In the original PaDIS, the data-consistency correction at each timestep is applied and immediately thrown away. The modification in `inverse_nodist.py` captures these corrections (measurement updates) and writes them to `measurement_updates/`. The KLIP notebook then treats them as a matrix and applies an eigendecomposition to separate the signal subspace from noise — improving reconstruction quality and enabling unsupervised anomaly scoring without any additional trained model.
