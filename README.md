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

## Project containers

To avoid reinstalling an environment on your different machines, we provide docker images for the various release (e.g., `v2026.1`).

### Using Docker (local machine)

Move to the directory `docker_images` and run

```bash
docker build \
    --tag cloe-org:<version_tag> \
    --file Dockerfile \
    .
```

After the image has built successfully, you can run the container interactively using

```bash
docker run --rm -it \
    --name cloe-org \
    cloe-org:<version_tag> \
    bash
```

You can verify the correct installation of the cloe environement in the container with

```bash 
python -c "import cloelib, cloelike, darkflipper, cloerun; from importlib.metadata import version; [print(p, version(p)) for p in ('cloelib', 'cloelike', 'darkflipper', 'cloerun')]"
python -c "import HMcode2020Emu; print('HMcode2020Emu OK')"
cloerun --help
```

It is possible to run the image using the GPU available on the hardware with

```bash
docker run --rm -it \
  --gpus all \
  --name cloe-org \
  cloe-org:<version_tag> \
  bash
```

### Using Apptainer (on HPC clusters)

HPC clusters usually have a different version of docker installed called `Apptainer` or `Singularity`. We recommand to check the documentation of your HPC cluster to now which software is used. Building and executing code in the container is slightly different than Docker but follows the same logic: build the image, then execute your code in it.

To build your image on the HPC cluster you can either pull it from an online server or build it locally with docker before copying it on the HPC cluster using

```bash
docker save -o cloe-org-<version_tag>.tar cloe-org:<version_tag>
gzip cloe-org-<version_tag>.tar cloe-org-<version_tag>.tar.gz
scp cloe-org-<version_tag>.tar.gz <name_of_the_cluster>:path/to/your/container/directory
```

You can then build using `singularity` or `apptainer` depending on the HPC. The following line is written with `singularity` but the same can be run replacing with `apptainer`. Leonardo HPC cluster relies on `singularity`.

```bash
cd path/to/your/container/directory
srun --mem=16G --cpus-per-task=4 --time=02:00:00 --pty bash
export singularity_TMPDIR=$WORK/path/to/singularity_tmp
export singularity_CACHEDIR=$WORK/path/to/singularity_cache
mkdir -p $SINGULARITY_TMPDIR $SINGULARITY_CACHEDIR
singularity build cloe-org-2026.1.sif docker-archive://cloe-org-2026.1.tar
```

After the image built, you can test it running the following

```bash
singularity shell cloe-org-2026.1.sif #creates a shell running in the container

#Run in the container
python -c "import cloelib, cloelike, darkflipper, cloerun; from importlib.metadata import version; [print(p, version(p)) for p in ('cloelib', 'cloelike', 'darkflipper', 'cloerun')]"
python -c "import HMcode2020Emu; print('HMcode2020Emu OK')"
cloerun --help
```

### Binding/Mounting folders

When you run on the container, you run in the virtual environment so you do not have access to the folders available on your local computer or the HPC clusters. Before launching the container, you need to bind the relevant folders.

```bash
# When using Docker
docker run --rm --it \
    --mount type=bind,source=/path/to/your/local/folder,target=/path/in/the/container \
    --name cloe-org \
    cloe-org:<version_tag> \
    bash

#When using singularity
singularity shell \
    --bind /path/to/your/local/folder:/path/in/the/container \
    cloe-org-<version_tag>.sif
```
