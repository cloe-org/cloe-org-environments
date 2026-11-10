FROM python:3.12-slim-bookworm AS base

ENV SHELL=/bin/bash \
    QT_QPA_PLATFORM=offscreen \
    PIP_NO_CACHE_DIR=1 \
    DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    # Make the image well-behaved on read-only filesystems (apptainer/SIF):
    #   UV_NO_SYNC      — `uv run` skips the auto-sync that would otherwise
    #                     mutate /app/.venv. Run with what's baked in.
    #   UV_CACHE_DIR    — uv's package cache. /tmp is tmpfs, writable in
    #                     read-only SIFs and writable sandboxes alike.
    #   COVERAGE_FILE   — pytest-cov writes its data file here instead of
    #                     /app/.coverage; needed because pyproject sets
    #                     `addopts = --cov=shapepipe` and pytest-cov erases
    #                     the file at startup.
    UV_NO_SYNC=1 \
    UV_CACHE_DIR=/tmp/uv-cache \
    COVERAGE_FILE=/tmp/.coverage

# System dependencies — three categories:
RUN apt-get update -y --quiet && \
    apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        gfortran \
        git \
        wget \
        pkg-config \
        libfftw3-dev libfftw3-bin \
        libgsl-dev \
        libcfitsio-dev \
        libproj-dev proj-bin \
        libgl1-mesa-glx \
        libgfortran5 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

    # OpenMPI from source — required for hybrid Apptainer MPI on HPC clusters.
ARG OMPI_VERSION=5.0.8
ARG OMPI_SERIES=v5.0
RUN cd /tmp && \
    wget -q "https://download.open-mpi.org/release/open-mpi/${OMPI_SERIES}/openmpi-${OMPI_VERSION}.tar.bz2" && \
    tar xjf "openmpi-${OMPI_VERSION}.tar.bz2" && \
    cd "openmpi-${OMPI_VERSION}" && \
    ./configure --prefix=/opt/ompi \
        --with-pmix=internal --with-prrte=internal \
        --with-hwloc=internal --with-libevent=internal \
        --disable-dlopen --disable-sphinx && \
    make -j"$(nproc)" && make install && \
    cd / && rm -rf /tmp/openmpi-*
ENV PATH="/opt/ompi/bin:${PATH}" \
    LD_LIBRARY_PATH="/opt/ompi/lib:${LD_LIBRARY_PATH}"

WORKDIR /app

FROM base AS cloe-org
LABEL description="Conda-free cloe-org environment to run chains"

# Interactive tools for working inside the container.
RUN apt-get update -y --quiet && \
    apt-get install -y --no-install-recommends \
        vim \
        less \
        tmux \
        htop \
        procps \
        ripgrep \
        fd-find \
        jq \
        bat \
        curl \
        ca-certificates \
        git-lfs \
        rsync \
        unzip zip \
        openssh-client \
        locales && \
    if command -v batcat >/dev/null; then ln -sf "$(command -v batcat)" /usr/local/bin/bat; fi && \
    if command -v fdfind >/dev/null; then ln -sf "$(command -v fdfind)" /usr/local/bin/fd; fi && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# liblapack-dev: cosmosis's MultiNest links -llapack, and the base image ships
RUN apt-get update -y --quiet --fix-missing && \
    apt-get dist-upgrade -y --quiet --fix-missing && \
    apt-get install -y --quiet \
        autoconf \
        automake \
        libtool \
        pkg-config \
        git \
        htop \
        npm \
        tmux \
        liblapack-dev \
        libgsl-dev \
        libcfitsio-dev \
        libfftw3-dev \
        perl \
        curl \
        ghostscript \
        libboost-math-dev && \
    ln -sf "$(command -v gcc)" /usr/local/bin/gcc-15 && \
    ln -sf "$(command -v g++)" /usr/local/bin/g++-15 && \
    rm -rf /var/lib/apt/lists/*

# TinyTeX pinned to a TeX Live year (frozen tlnet-final mirror); bump both once a year.
# The packages serve matplotlib's usetex figures; sfmath gives them sans-serif
# maths (`\usepackage[cm]{sfmath}`).
ENV TEXLIVE_YEAR=2025 \
    TINYTEX_VERSION=2026.02 \
    TINYTEX_DIR=/opt \
    PATH=/opt/.TinyTeX/bin/x86_64-linux:$PATH
RUN set -eux; \
    curl -fsSL https://yihui.org/tinytex/install-bin-unix.sh | sh; \
    tlmgr option sys_bin /usr/local/bin; \
    tlmgr option repository \
      "https://ftp.math.utah.edu/pub/tex/historic/systems/texlive/${TEXLIVE_YEAR}/tlnet-final/"; \
    tlmgr option docfiles 0; \
    tlmgr option srcfiles 0; \
    tlmgr install \
        type1cm \
        cm-super \
        dvipng \
        underscore \
        ulem \
        amsmath \
        amsfonts \
        geometry \
        sfmath \
        xcolor; \
    tlmgr path add; \
    latex --version >/dev/null; dvipng --version >/dev/null

# Install python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir \
    # python runtime
    setuptools \
    wheel \
    # Interactive and debugging tools
    ipython \
    ipykernel \
    jupyter \
    # Plotting
    matplotlib \
    seaborn \
    # Euclid and project-specific packages
    euclidlib \
    nautilus-sampler==1.0.6 \
    tf-keras==2.20.* \
    pre-commit 

# Install the cloe-org dependencies
RUN pip install --no-cache-dir \
    "cloelib[camb,classy,comet-emu,euclidemu2,pylevin,mpmath,tensorflow,tf-keras,cosmopower-jax,baccoemu,hiclassy,emantis,pbjcosmo,fastpt,react,mgclassy,FlamingoBaryonResponseEmulator] @ git+https://github.com/cloe-org/cloelib.git@v0.11" \
    "HMcode2020Emu @ git+https://github.com/MariaTsedrik/HMcode2020Emu.git" \
    "MGemu @ git+https://github.com/nebblu/MGEmus.git" \
    "mochi_classy @ git+https://github.com/mcataneo/mochi_class_public.git@mochi_classy" \
    cloelike==0.3 \
    darkflipper \
    "cloerun[analyse] @ git+https://github.com/cloe-org/cloerun.git@main" 

# Download the HMCode 2020 emulator to avoid writing it later.
RUN python -c "import HMcode2020Emu"

# For workflow management
RUN pip install --no-cache-dir \
    "snakemake==8.30.0"

