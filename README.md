# ztf-dust

Scripts to reproduce the results and figures of:

> Giunta, Karchev & Trotta (2026), *The colour variability of low-z SNe Ia is entirely explained by dust*
>
> [![arXiv](https://img.shields.io/badge/arXiv-2606.06593-b31b1b.svg)](https://arxiv.org/abs/2606.06593)

The analysis uses the [simplebayesn](https://github.com/marco-giunta/simplebayesn) package, which must be installed first.

---

## Dependencies

Install `simplebayesn` and its dependencies:
```bash
pip install git+https://github.com/marco-giunta/simplebayesn.git
```

Additional packages needed for the Foundation SALT2 fits:
```bash
pip install sncosmo tqdm
```

---

## Data

- **ZTF DR2**: download from [https://ztfcosmo.in2p3.fr/](https://ztfcosmo.in2p3.fr/)
- **Foundation DR1**: clone from [https://github.com/djones1040/Foundation_DR1](https://github.com/djones1040/Foundation_DR1)
- SALT2.JLA-B14 model files: available from the [SNANA](https://github.com/RickKessler/SNANA) distribution under `snana/models/SALT2/SALT2.JLA-B14`

---

## Workflow

The scripts are listed below in the order they should be run. Each script accepts `--input` / `--output` arguments; run any script with `--help` for details.

### 1. Foundation SALT2 fits

The official Foundation DR1 release only provides SALT2 fits for the cosmological subsample. This script recomputes fits for both the cosmo and no-cosmo samples using `sncosmo` with the `SALT2.JLA-B14` model, matching the original SNANA setup:

```bash
python foundation_salt_fit.py \
    --input /path/to/Foundation_DR1/ \
    --output data/foundation_salt.csv \
    --salt /path/to/SALT2.JLA-B14/
```

### 2. Gibbs sampling

**ZTF HQ VL (full sample, no colour cut):**
```bash
python ztf_gibbs.py \
    --input data/ztf_dr2.csv \
    --output chains/ztf_gibbs.h5
```

**ZTF HQ VL with $|c| \leq 0.3$ colour cut:**
```bash
python ztf_03_color_cuts_gibbs.py \
    --input data/ztf_dr2.csv \
    --output chains/ztf_03_gibbs.h5
```

**Foundation (full and cosmo-only):**
```bash
python foundation_gibbs.py \
    --input data/foundation_salt.csv \
    --output chains/foundation_gibbs.h5

python foundation_cosmo_gibbs.py \
    --input data/foundation_salt.csv \
    --output chains/foundation_cosmo_gibbs.h5
```

**ZTF subsamples split by host-galaxy environment** (stellar mass and DLR):
```bash
python ztf_gibbs_mass_splits.py \
    --input_data data/ztf_dr2.csv \
    --input_global data/ztf_globalhost.csv \
    --input_local data/ztf_localhost.csv \
    --output chains/splits/
```

### 3. Selection bias simulation

These scripts demonstrate the bias induced by unmodelled colour cuts, using simulated ZTF-like data:

```bash
# Gibbs sampler on simulated data with and without cuts
python ztf_sim_sel_gibbs.py \
    --input data/ztf_dr2.csv \
    --output chains/ztf_sim_gibbs.h5

# emcee sampler with likelihood renormalisation on the cut simulated data
python ztf_sim_sel_emcee.py \
    --input data/ztf_dr2.csv \
    --output chains/ztf_sim_emcee.h5
```

### 4. Figures and tables

Each script reads one or more chain `.h5` files and saves its output. Run with `--help` for argument details.

| Script | Output |
|---|---|
| `ztf_foundation_mb_c.py` | Fig. 1: observed magnitude-colour distributions of ZTF and Foundation cosmo/no-cosmo |
| `ztf_sim_sel_cornerplot.py` | Fig. 2: selection bias simulation cornerplot |
| `ztf_03_color_cuts_gibbs_posterior_cornerplot.py` | Fig. 3: ZTF colour parameter posteriors with/without cut |
| `ztf_gibbs_extrinsic_latent_population_plot.py` | Fig. 4: ZTF latent colour-magnitude distribution |
| `ztf_foundation_gibbs_posterior_cornerplot.py` | Fig. 5 and Fig. A2: ZTF vs Foundation posteriors |
| `ztf_gibbs_latents_plot.py` | Figs. 6-7: ZTF per-SN latents vs host environment |
| `ztf_gibbs_mass_splits_posterior_cornerplots.py` | Figs. 8-9: ZTF split-sample posteriors |
| `ztf_gibbs_posterior_cornerplot.py` | Fig. A1: full ZTF parameter cornerplot |
| `ztf_fnd_gibbs_table.py` | Tab. 3: parameter table |

---

## Citation

If you use these scripts, please cite:

> Giunta, Karchev & Trotta (2026), *The colour variability of low-z SNe Ia is entirely explained by dust*
