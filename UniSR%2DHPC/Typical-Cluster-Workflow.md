# Typical Cluster Workflow

This page explains the **typical end-to-end workflow** for running code on an HPC cluster (including UniSR resources and larger external centers like **CINECA**).

If you are used to working on your laptop, the biggest mental shift is that on a cluster:

- You do **not** run heavy workloads directly on the login node.
- You **request** compute resources from a scheduler.
- You should make your software environment **reproducible** (containers or environment files).
- You should be intentional about **where data lives** (slow vs fast storage).

If you want deeper dives while reading:

- **SSH access:** [Login Guide](/UniSR%2DHPC/Login-instructions)
- **Storage tiers & staging:** [Cluster Storage Architecture & Data Workflow](/UniSR%2DHPC/Cluster-Storage-Architecture-&-Data-Workflow)
- **SLURM basics:** [SLURM Cheatsheet](/UniSR%2DHPC/SLURM/SLURM-%2D-Cheatsheet) and [SLURM Best Practices](/UniSR%2DHPC/SLURM/SLURM-Best-Practices)
- **SLURM + Apptainer (mental model):** [SLURM and Apptainer for Dummies](/UniSR%2DHPC/SLURM-and-Apptainer-for-dummies)
- **Containers & Apptainer:** [Containers for Dummies](/UniSR%2DHPC/Apptainer/Containers-for-dummies), [Apptainer Cheatsheet](/UniSR%2DHPC/Apptainer/Apptainer-Cheatsheet), [Containers Best Practices](/UniSR%2DHPC/Apptainer/Containers-best-practices)

[[_TOC_]]

---

## The Core Abstraction (Slurm vs Apptainer)

Think of cluster work as two separate questions:

1. **Where will my code run?** (the *hardware*)
2. **In what environment will it run?** (the *software/OS*)

### Slurm Workload Manager = the hardware

**Slurm represents the machine your job runs on.**

- You ask Slurm for resources (CPUs, GPUs, RAM, time).
- Slurm decides *which node* you get and *when* you can run.
- Commands like `srun` and `sbatch` are “hardware requests”.

### Apptainer (or Docker) = the software / OS

**Apptainer represents the operating system + software stack your code runs inside.**

- A container image (e.g., `something.sif`) is a self-contained environment.
- It defines your OS userspace, libraries, Python version, and tools.
- Commands like `apptainer shell` / `apptainer exec` are “run inside this OS”.

### The key idea

**Slurm provides the computer. Apptainer provides the operating system.**

Most real workflows combine both:

- Slurm allocates a node with the resources you need.
- Apptainer runs your command inside the software environment you defined.

---

## Step 1 — Access & Setup

### 1. SSH into the cluster

From your local machine:

```bash
ssh surname.name@hsr.it@10.64.79.72
```

> If you need SSH configuration tips (keys, config file, etc.), see the [Login Guide](/UniSR%2DHPC/Login-instructions).

### 2. Create a project folder and clone your repository

A common pattern is to keep **code in your Home directory** (small, backed up, always available), and keep **large datasets** on shared storage.

```bash
mkdir -p ~/projects
cd ~/projects

git clone https://github.com/<org>/<repo>.git
cd <repo>
```

If you need to set up Git on the cluster (SSH keys, username/email), see:

- [Git configuration](/UniSR%2DHPC/Useful-tools/Git-configuration)

---

## Step 2 — Data Management

### 1. Bring data to the cluster

You generally have two ways to get data into the cluster.

#### Option A: Transfer from your laptop (SCP)

```bash
# From your laptop
scp -r ./my_dataset surname.name@hsr.it@10.64.79.72:/mnt/data/unisr-data/datasets/my_dataset
```

#### Option B: Download from the internet (wget/curl)

```bash
# On the cluster
mkdir -p /mnt/data/unisr-data/datasets/some_dataset
cd /mnt/data/unisr-data/datasets/some_dataset

wget https://example.org/data.zip
# or
curl -L -o data.zip https://example.org/data.zip
```

### 2. Storage strategy (slow vs fast)

Clusters usually expose multiple storage tiers.

- **“Slow” storage** (network storage, high capacity) is best for:
  - initial uploads
  - long-term archiving
  - data exploration that is not I/O intensive

- **“Fast” storage** (parallel FS / SSD tier) is best for:
  - training jobs
  - heavy data loaders
  - large-scale preprocessing

For UniSR specifically, a typical mapping is:

- **Slow / landing zone:** `/mnt/data/unisr-data`
- **Fast (I/O) for active runs:** `/mnt/beegfs/scratch/unisr-data` (SSD tier)
- **Fast-ish for very large datasets:** `/mnt/beegfs/data/unisr-data` (HDD tier)

A common staging pattern is:

```bash
# Stage data from slow to fast before training
cp -r /mnt/data/unisr-data/datasets/my_dataset \
      /mnt/beegfs/scratch/unisr-data/datasets/my_dataset
```

> For more details (and the rationale behind each path), see [Cluster Storage Architecture & Data Workflow](/UniSR%2DHPC/Cluster-Storage-Architecture-&-Data-Workflow).
>
> Note: Some parallel filesystems may behave poorly from login nodes. If copying feels slow or hangs, do the staging step from inside an interactive allocation.

---

## Step 3 — Environment Definition

The goal is to define a **reproducible** software environment that you can run on:

- UniSR cluster
- other clusters (e.g., CINECA)
- your laptop (for debugging)

### Container recipes: Apptainer definition file or Dockerfile

You typically describe your environment as a **recipe**:

- **Apptainer definition file** (often named `container.def` or `Singularity.def`)
- **Dockerfile**

In that recipe, you define things like:

- **Python version** (e.g., 3.12)
- libraries (e.g., `pandas`, `matplotlib`, `torch`)
- system packages (e.g., `git`, `ffmpeg`, `libgl1`)
- your preferred package manager:
  - Conda / Miniconda
  - Mamba
  - uv
  - plain `python -m venv`

If containers are new to you, start here:

- [Containers for Dummies](/UniSR%2DHPC/Apptainer/Containers-for-dummies)

If you already know containers, these are the practical references you’ll use most:

- [Apptainer Cheatsheet](/UniSR%2DHPC/Apptainer/Apptainer-Cheatsheet) (binds, overlays, exec/shell)
- [Containers Best Practices](/UniSR%2DHPC/Apptainer/Containers-best-practices) (how to keep images clean and purpose-aligned)

### A minimal Apptainer definition example

```def
Bootstrap: docker
From: ubuntu:24.04

%post
    apt-get update && apt-get install -y --no-install-recommends \
        python3 python3-venv python3-pip git ca-certificates \
        && rm -rf /var/lib/apt/lists/*

    python3 -m venv /opt/venv
    . /opt/venv/bin/activate
    pip install --upgrade pip
    pip install pandas matplotlib torch

%environment
    export PATH="/opt/venv/bin:$PATH"

%runscript
    exec "$@"
```

### A minimal Dockerfile example

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 python3-venv python3-pip git ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /opt/venv \
 && . /opt/venv/bin/activate \
 && pip install --upgrade pip \
 && pip install pandas matplotlib torch

ENV PATH="/opt/venv/bin:$PATH"

CMD ["python3", "--version"]
```

---

## Step 4 — Build the Image (Security Warning)

A recipe file (Apptainer definition file / Dockerfile) is **not** an environment by itself.

- It is a **build recipe**.
- You must build it into a **container image**:
  - Docker image (e.g., `myproj:latest`)
  - Apptainer image (e.g., `myproj.sif`)

### ⚠️ Important: don’t build images on the cluster

**Avoid building container images directly on the cluster.**

- Building often requires elevated privileges or complex build tooling.
- Many centers explicitly restrict builds on shared infrastructure.
- It increases security risk and can create performance issues for other users.

### Recommended approach

Pick one of these patterns:

1. **Build locally, then transfer the built image**
2. **Use an existing Docker image and convert it to Apptainer**

#### Option A: Build locally and copy the image to the cluster

Build a Docker image on your laptop/workstation:

```bash
docker build -t myproj:latest .
```

Then convert to Apptainer (on a machine where you are allowed to do so):

```bash
apptainer build myproj.sif docker-daemon://myproj:latest
```

Transfer the `.sif` to the cluster (store images on shared storage):

```bash
scp myproj.sif surname.name@hsr.it@10.64.79.72:/mnt/data/unisr-data/images/myproj.sif
```

#### Option B: Convert directly from a registry image

If your image is on Docker Hub (or another registry):

```bash
apptainer build myproj.sif docker://pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime
```

Then copy `myproj.sif` to the cluster as above.

> Some clusters require that image builds happen off-cluster or in dedicated build services. When in doubt, follow the center’s policy.

Related reading:

- [Containers Best Practices](/UniSR%2DHPC/Apptainer/Containers-best-practices)
- [Apptainer Cheatsheet](/UniSR%2DHPC/Apptainer/Apptainer-Cheatsheet)

---

## Step 5 — Job Execution (Slurm + Apptainer together)

This is the “payoff” stage:

- **Slurm** gives you the allocated compute node(s).
- **Apptainer** runs your code inside the environment.

### Interactive mode (good for debugging)

Use this when you want a shell on a compute node to iterate quickly.

For development workflows (Jupyter / VSCode on a compute node inside Apptainer), see:

- [Development tools](/UniSR%2DHPC/Development-tools)

1) Request resources with `srun`:

```bash
# Example: request 1 GPU for 2 hours
srun --partition=interactive --gres=gpu:1 --cpus-per-task=4 --mem=16G --time=02:00:00 --pty bash
```

More `srun` / partition options:

- [SLURM Cheatsheet](/UniSR%2DHPC/SLURM/SLURM-%2D-Cheatsheet)
- [SLURM Best Practices](/UniSR%2DHPC/SLURM/SLURM-Best-Practices)

2) Once you are on the compute node, run your container:

```bash
# Shell inside container
apptainer shell --nv \
  --bind "$HOME/projects/<repo>:/workspace" \
  --bind /mnt/beegfs/scratch/unisr-data:/mnt/beegfs/scratch/unisr-data \
  /mnt/data/unisr-data/images/myproj.sif
```

Or run a single command:

```bash
apptainer exec --nv \
  --bind "$HOME/projects/<repo>:/workspace" \
  /mnt/data/unisr-data/images/myproj.sif \
  python -u /workspace/train.py --data /mnt/beegfs/scratch/unisr-data/datasets/my_dataset
```

### Batch mode (recommended for real runs)

Batch jobs are the standard HPC approach:

- reproducible
- schedulable
- easy to log
- you can disconnect and the job keeps running

1) Create a submission script, e.g. `train.sbatch`:

```bash
#!/usr/bin/env bash
#SBATCH --job-name=my-train
#SBATCH --partition=cuda
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=2-00:00:00
#SBATCH --output=logs/%x-%j.out
#SBATCH --error=logs/%x-%j.err

set -euo pipefail

mkdir -p logs

IMAGE=/mnt/data/unisr-data/images/myproj.sif
REPO_DIR=$HOME/projects/<repo>

# Optional: stage data to fast storage (do this only if needed)
# cp -r /mnt/data/unisr-data/datasets/my_dataset /mnt/beegfs/scratch/unisr-data/datasets/

apptainer exec --nv \
  --bind "$REPO_DIR:/workspace" \
  --bind /mnt/beegfs/scratch/unisr-data:/mnt/beegfs/scratch/unisr-data \
  "$IMAGE" \
  python -u /workspace/train.py \
    --data /mnt/beegfs/scratch/unisr-data/datasets/my_dataset \
    --out  /mnt/beegfs/scratch/unisr-data/experiments/$USER/myproj/$SLURM_JOB_ID
```

2) Submit it:

```bash
sbatch train.sbatch
```

3) Monitor it:

```bash
squeue -u "$USER"
```

Tips that help a lot in practice:

- Keep a persistent terminal session with [Terminal multiplexers](/UniSR%2DHPC/Useful-tools/Terminal-multiplexers) (e.g., tmux) for monitoring/debugging.
- Make it obvious where you are (login vs compute vs container) with [Useful .bashrc settings](/UniSR%2DHPC/Useful-tools/Useful-.bashrc-settings).

---

## What changes when you move to bigger centers (e.g., CINECA)?

The **workflow stays the same**, but details vary:

- Slurm options / partitions have different names.
- Filesystem paths differ (and quota policies are usually stricter).
- Container policies can be more restrictive (especially builds).

If you keep these invariants, your work transfers cleanly:

- **Code** in Git
- **Environment** in a container recipe + built image
- **Jobs** as `sbatch` scripts
- **Data** staged intentionally to the right storage tier
