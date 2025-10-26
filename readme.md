# Galaxy-Reconstruction-ISTA

Reconstruct spiral-galaxy images from **sparse, noisy** measurements using **Iterative Soft-Thresholding (ISTA)** with DCT/DWT sparsity. Images come from **DESI Legacy Imaging Surveys (DECaLS)**.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue) ![Status](https://img.shields.io/badge/Method-ISTA-black) ![License](https://img.shields.io/badge/License-MIT-green)

---

## Overview

We solve a LASSO problem for a sparse representation (x) of an image in a transform basis (W):

[
\min_{x}\ \tfrac12|A W^\top x - b|_2^2 + \lambda |x|*1,
\quad
\text{ISTA:}\ \ x^{k+1}=\mathcal{S}*{\alpha\lambda}\Big(x^k-\alpha W A^\top(A W^\top x^k-b)\Big),
]

where (A) is a masking/subsampling (and noise) operator, (b) the observed data, (\mathcal{S}) soft-thresholding, and (W) is **DCT** or **DWT** (PyWavelets). We compare bases, quantify reconstruction quality, and report sparsity.

---

## Requirements

* numpy, scipy, **pywt**, **h5py**, tqdm, matplotlib
  Optional (recommended): scikit-image (PSNR/SSIM), jupyter, pytest

Install:

```bash
pip install -r requirements.txt
# or
pip install numpy scipy pywt h5py tqdm matplotlib scikit-image jupyter pytest
```

---

## Project Structure

```
Galaxy-Reconstruction-ISTA/
├─ data/                      # raw/processed DECaLS samples (not committed)
├─ notebooks/
│  └─ ista_galaxy_demo.ipynb  # end-to-end walkthrough
├─ src/
│  ├─ io_utils.py             # load/select/preprocess galaxy tiles
│  ├─ ops.py                  # A, AT, masking, noise, subsampling
│  ├─ transforms.py           # DCT/DWT forward/inverse wrappers
│  ├─ ista.py                 # soft-threshold, single-step, full loop
│  ├─ metrics.py              # MSE, PSNR, SSIM, sparsity
│  └─ viz.py                  # side-by-side plots
├─ scripts/
│  └─ run_ista.py             # CLI for batch runs
├─ results/                   # figures and CSV logs
└─ README.md
```

---

## Quickstart

```bash
git clone https://github.com/<you>/Galaxy-Reconstruction-ISTA.git
cd Galaxy-Reconstruction-ISTA

# (optional) env
python -m venv .venv && source .venv/bin/activate

pip install -r requirements.txt

# Notebook demo
jupyter lab notebooks/ista_galaxy_demo.ipynb
```

CLI example:

```bash
python scripts/run_ista.py \
  --input data/decals_sample.h5 \
  --image-id 12345 \
  --basis dwt --wavelet db4 \
  --mask-rate 0.3 --noise-sigma 0.02 \
  --lambda 0.01 --steps 500 --step-size 1.0 \
  --out results/run_db4_30mask.json
```

---

## Pipeline (what actually happens)

1. **Load & Prep**

   * Read DECaLS cutout (HDF5/NumPy). Normalize to ([0,1]). Optional center-crop.
   * Add **Gaussian noise** ((\sigma)) and **random subsampling** (mask rate (p)).

2. **Transforms**

   * Choose **DCT** or **DWT** (`pywt`), implement forward/inverse (W, W^\top).

3. **ISTA Core**

   * Soft-threshold: (\mathcal{S}_\tau(u)=\text{sign}(u)\max(|u|-\tau,0)).
   * Single step + loop with stopping by **relative decrease** or max steps.
   * Step size (\alpha \le 1/L) with (L=|A W^\top|^2) (use a conservative (\alpha) if you can’t estimate (L)).

4. **Reconstruction**

   * Recover image (\hat{y}=W^\top x^{(K)}).
   * Plot **original / noisy / masked / reconstructed** grids.

5. **Analysis**

   * **Sparsity** = non-zeros in (x).
   * **PSNR/SSIM/MSE** vs. ground truth.
   * Compare **DCT vs DWT**; sweep (\lambda), step size, iterations.

---

## Results

You must report numbers, not vibes:

* **Reconstruction error**: MSE/PSNR/SSIM.
* **Sparsity**: (|x|_0) (count of non-zero coefficients).
* **Ablations**: basis (DCT/DWT), mask rate, (\lambda), iterations.

Store CSV logs in `results/` and include the best/worst examples as PNGs. If you only show pretty pictures, it’s not science.

---

## Repro Tips (so reviewers can’t roast you)

* Fix RNG seeds for noise/masks.
* Log every hyperparameter: mask rate, (\sigma), (\lambda), steps, (\alpha), basis, wavelet levels.
* Sanity checks: with **no masking & zero noise**, ISTA should converge to identity quickly.

---

## Citation & Data

* DESI Legacy Imaging Surveys (DECaLS). Cite the survey per their guidelines.
* If you release sample cutouts, ensure license compliance.

---

## License

MIT. 
