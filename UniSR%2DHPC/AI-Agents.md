# AI Agents on the Cluster

AI coding agents (e.g. Claude Code) can run directly on the cluster login node and operate on your behalf: write code, submit SLURM jobs, inspect logs, etc. Since the cluster is a shared environment, agents must follow the same usage rules as human users.

To make this easy, we maintain a shared set of **agent rules** that tell an agent how to behave on the cluster.

[[_TOC_]]

---

# The agent-rules repository

The rules are kept in a dedicated repository: https://github.com/marcizzzsr/unisr-agent-rules

A stable copy is available on the cluster at `/mnt/data/unisr-data/agent-rules`.

The rules cover:
- **Golden rule**: never run compute on the login node. Every workload (including downloads, installs and data copies) runs on a compute node allocated through SLURM.
- **Running jobs**: SLURM + Apptainer by default, SLURM + venv (e.g. `uv`) when a container is overkill. Partitions, required `sbatch` options, GPU requests with `--gres`.
- **Apptainer images**: never build on the cluster, look for shared images in `/mnt/data/unisr-data/apptainer_images/images` first.
- **Data and storage**: where data lives and how to stage it from slow to fast storage.
- **Experiment tracking**: log experiments to the on-premise [MLflow](/UniSR%2DHPC/MLflow/DS-User-Guide) server.
- **User interaction**: when the agent must inform you (writes to shared folders, long runs, use of pre-built images) and when it must wait for your confirmation (data moves between storage tiers, any deletion in shared folders).

---

# Claude Code

## Installation

Install Claude Code in your home directory by running the following command on the cluster:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Then start it with `claude` from your project folder.

## Enable the cluster rules

Claude Code automatically loads instructions from `CLAUDE.md` files at the start of every session. To enable the cluster rules, add the following line to your `~/.claude/CLAUDE.md` (create the file if it doesn't exist):

```
@/mnt/data/unisr-data/agent-rules/CLAUDE.md
```

The `@` syntax imports the shared file, so you will always get the latest version of the rules. You can add your own personal preferences below that line.

---

# Codex

## Installation

Install Codex in your home directory by running the following command on the cluster:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Then start it with `codex` from your project folder.

## Cluster rules

Rules for Codex are not available yet: the agent-rules repository currently provides only the Claude Code version (`CLAUDE.md`).

---

# Notes
- **Do not edit the shared copy in `/mnt/data/unisr-data/agent-rules` in place.** Propose changes on the repository instead; the cluster copy is updated with `git pull`.
- The rules guide the agent's behavior but they are not a technical enforcement: always review what your agent does, especially in autonomous modes.
