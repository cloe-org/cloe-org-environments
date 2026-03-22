# 🗂️ cloe-org-environments

**Unified Mamba/Conda environments for the entire `cloe-org` ecosystem.**

This repository provides **versioned, ready-to-use Conda/Mamba environment specifications** so you can get started immediately.

---

> [!WARNING]
> 🟧 **Important: Conda vs Mamba Usage**
>
> We strongly recommend using **mamba** (The Fast Cross-Platform Package Manager) due to recent changes in the **Anaconda Terms of Service (ToS)** 🐍.
>
> If you use `conda` with the default Anaconda channels, your organization **may incur licensing fees** if it has more than 200 employees.
>
> 🔗 More info: https://www.fz-juelich.de/en/rse/the_latest/the-anaconda-is-squeezing-us

---

While primarily targeting speed and reliability, this environment manager uses **conda-forge as the default channel**.

This community-driven channel is independent of Anaconda, avoiding any ToS issues while preserving the powerful features of the Conda ecosystem.


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
