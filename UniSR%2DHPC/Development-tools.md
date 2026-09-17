# Development Environment Launchers

This guide provides a unified overview of a suite of scripts that launch common development environments inside an **Apptainer** container within an HPC environment using **SLURM**.

These tools provide a convenient wrapper to get a fully functional development environment running on a compute node, accessible from your local machine.

> **You are highly encouraged to check the official github repository of the full collection of these tools that can be found [here](https://github.com/AI-UniSR/cluster-entrypoints) along with custom documentation files for each tool implemented. You can find a stable version of the repository in the cluster at `/mnt/data/unisr-data/entrypoints`**

---

## Common Structure

All tools share the same file structure:

```
.
├── wrapper.sh   # Entry point — parses arguments and submits the SLURM job
├── job.sh       # Executed on the compute node to start the service
└── *.md         # Documentation for the specific tool
```

## How It Works: A General Overview

Each tool follows the same core process:

1.  **First-Time Setup**: On the first run, the script will perform a brief setup and add a unique alias to your `~/.bashrc` file. This means you only need to use the full script path once; afterward, you can use the simple alias.
2.  **`wrapper.sh`**: This is the entry point. It validates your input arguments, parses any extra bind mounts, and launches an interactive job on a compute node using SLURM (`srun`).
3.  **`job.sh`**: This script runs on the compute node. It prepares persistent directories for user data, determines the node's IP address if needed, and finally executes the appropriate service inside an Apptainer container with your specified directories mounted.

## Common Usage
First you either need to clone the [original repository](https://github.com/AI-UniSR/cluster-entrypoints) to your home directory or access a stable version at `/mnt/data/unisr-data/entrypoints`. From there, locate the tool you want to use and `cd` into its dedicated directory.

Tools are launched using a similar command format:

```bash
<tool-executable> <host-workdir> [--bind <host:container> ...]
```

### Parameters

*   `<host-workdir>` — **Required**.
    The directory on the host machine that you want to mount as your main project folder inside the container.
*   `--bind <host:container>` — **Optional**.
    Use this to specify additional directories to mount inside the container. This argument is passed directly to Apptainer, and you can use it multiple times.

### Example

This example mounts a project directory and two additional directories for datasets and tools. This command format works for the tools described below.

```bash
./code-server/wrapper.sh ~/projects/mycode \
   --bind /mnt/data/datasets:/workspace/datasets \
   --bind /mnt/tools:/workspace/tools
```

## 🧩 General Notes

*   The main development directory (`<host-workdir>`) must exist before running the script.
*   User configurations, extensions, and data are persisted in a hidden directory in your home (e.g., `~/.vscode-tunnel` or `~/coder`). Do not delete these if you wish to maintain your settings across sessions.

---

[[_TOSP_]]
