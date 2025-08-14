# Robust Air Pollutant Forecasting with Time‑Series Transformers

This repository contains the notebook and materials for a comparative study of **Time‑Series Transformers (TST)** — **Informer**, **PatchTST**, and **Crossformer** — against an **N‑BEATS** baseline for forecasting air pollutants (NO, NO₂, NOx, O₃, PM₂.₅) under **limited‑data** constraints.

We evaluate models on two climatically distinct regions in Israel (Mediterranean vs. desert), using a **single monitoring station per region**. All models share a unified pipeline: **120‑hour look‑back** → **24‑hour forecast**, repeated over a **7‑day** test horizon, and are compared with **RRMSE** as the primary metric.

> Notebook: `# Time Series Transformers Fiinal Project - Dor Halevy,Sagiv Bar, Hadaer Mentel.ipynb`

---

## 🔧 Environment Setup

We recommend Python 3.10–3.11.

```bash
# 1) Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# 2) Upgrade pip and install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

> If you plan to use a GPU with PyTorch, follow the official install selector to match your CUDA version:
> https://pytorch.org/get-started/locally/

---

## 📦 Model Implementations

The study used public implementations for each architecture. Because some repos are **not pip‑installable** as packages, we recommend adding them as git submodules (or cloning into a `models/` folder) and importing from source:

```bash
mkdir -p models && cd models

# Crossformer (official)
git clone https://github.com/Thinklab-SJTU/Crossformer.git

# PatchTST (reference implementation)
git clone https://github.com/yuqinie/patchtst.git

# Informer (reference — often distributed within FEDformer forks or dedicated repos)
git clone https://github.com/zhouhaoyi/Informer2020.git

cd ..
```

Then, in scripts/notebooks, add the paths to `sys.path` before imports, e.g.:

```python
import sys, os
sys.path.append(os.path.join("models", "Crossformer"))
sys.path.append(os.path.join("models", "patchtst"))
sys.path.append(os.path.join("models", "Informer2020"))
```

> If your local code already contains adapted copies of these models, you can skip cloning and use your versions directly.

---

## 📁 Suggested Repo Structure

```
.
├── README.md
├── requirements.txt
├── notebooks/
│   └── # Time Series Transformers Fiinal Project - Dor Halevy,Sagiv Bar, Hadaer Mentel.ipynb
├── data/
│   ├── raw/               # original CSV/XLSX (not committed)
│   └── processed/         # aligned/cleaned hourly data
├── models/                # cloned model repos or adapted code
├── scripts/
│   ├── preprocess.py      # cleaning, alignment, interpolation
│   ├── train_eval.py      # shared training/eval loop
│   └── plot_results.py    # figures and tables
└── results/
    ├── predictions/       # per-model per-pollutant CSVs
    └── figures/           # RRMSE bars, runtime charts, etc.
```

---

## 🧪 Reproducing the Study (outline)

1. **Data preparation**  
   - Align meteorological + air‑quality data to hourly resolution (2020‑03‑09 → 2022‑12‑31).  
   - Apply a **3‑hour lag** to meteorology to sync with pollutant readings.  
   - Clean missing/invalid values and **Akima** interpolation where appropriate.  
   - Train/val/test split: **train to 2022‑06‑01**, **val = last 20% of train**, **test = final 7 days**.  
   - Feature scaling fitted **only on train**, applied to val/test.

2. **Windowing**  
   - Input: **120 hours**; Output: **next 24 hours** (rolling, 7 daily windows).

3. **Training**  
   - Loss: **MSE**, Optimizer: **Adam**.  
   - Early stopping (patience = 8) on validation loss.  
   - Fix seeds where supported.

4. **Evaluation**  
   - Primary metric: **Relative RMSE (RRMSE)**, averaged over the 7 daily windows per pollutant.  
   - Save predictions and ground truths for analysis; record training runtimes.

5. **Notes per model**  
   - **PatchTST**: univariate‑target runs (separate model per pollutant) in the common implementation.  
   - **Crossformer / Informer**: multivariate‑target runs (predict all 5 pollutants together).

---

## 📊 Key Findings (high‑level)

- **PatchTST** often yields the lowest RRMSE, especially for **O₃** and **PM₂.₅**.  
- **Crossformer** is typically second‑best in accuracy and **fastest to train**.  
- **Informer** underperforms on **NO** in our setup.  
- Across both regions, TSTs generally **outperform** the single **N‑BEATS** baseline; hybrid/ensemble approaches are a promising next step.

> See the notebook and figures in `results/` for full details.

---

## ▶️ Running the Notebook

```bash
jupyter notebook
# or
jupyter lab
```

Open the notebook in `notebooks/`, update paths to your `data/` and `models/` folders if needed, and run the cells top‑to‑bottom.

---

## 📝 Citations

If you use this work, please cite the underlying paper/summary in this repo and the original model repositories.(See the References section inside the notebook/paper for full details.)

---

## 📤 How to Publish to GitHub (Quick Steps)

1. Create a new repo on GitHub (public or private).  
2. In your project folder locally:
   ```bash
   git init
   git add .
   git commit -m "Initial commit: TST pollutant forecasting project"
   git branch -M main
   git remote add origin https://github.com/<YOUR_USERNAME>/<REPO_NAME>.git
   git push -u origin main
   ```
3. Verify that the notebook renders on GitHub and that `README.md` appears on the repo homepage.

---

## License

Specify your preferred license (e.g., MIT) in a `LICENSE` file.
