# 🗂️ cloe-org-environments

**Unified Mamba/Conda environments for the entire `cloe-org` ecosystem.**

This repository provides **versioned, ready-to-use Conda/Mamba environment specifications** so you can get started immediately.

We strongly recommend the use of mamba, The Fast Cross-Platform Package Manager, due to the drastic change in the Terms of Service (ToS) of Anaconda :snake:. 

If you use `conda` and default conda channels, you might incur in fees, if your organization has more than 200 employees.
See:  https://www.fz-juelich.de/en/rse/the_latest/the-anaconda-is-squeezing-us

While primarily targeting speed and reliability, this package/environment manager has conda-forge as default channel to pull the packages from. 
This community-driven channel is independent of Anaconda and avoids any ToS breaches, while keeping the great features of such a package/environment system based on conda.


---

## 🔧 Quick start

Clone this repository and create the environment directly from GitHub using either `conda` or `mamba` (we strongly recommend `mamba`):

```bash
mamba/conda env create -f ./environments/cloe-org-env-[VERSION].yaml
mamba/conda activate cloe-org-env[-VERSION]
```

Optionally, replace `[-VERSION]` with your desired release (e.g., `-v2026.1`), if the environment file you used contains a specific version name.
Default environment: `cloe-org-env`

---

## 📦 Project packages

Once the environment is activated, install `cloelib` with the required extras, along with `cloelike`, following the project-specific installation instructions.

---
