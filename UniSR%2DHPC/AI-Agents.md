# AI Agents on the Cluster

AI coding agents (e.g. Claude Code, Codex, OpenCode) can run directly on the cluster login node and operate on your behalf: write code, submit SLURM jobs, inspect logs, etc. Since the cluster is a shared environment, agents must follow the same usage rules as human users.

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


---

# Codex

## Installation

Install Codex in your home directory by running the following command on the cluster:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Then start it with `codex` from your project folder.


---

# OpenCode

[OpenCode](https://opencode.ai/) is an open-source AI coding agent that runs in your terminal, like Claude Code and Codex. Its key feature is that **it can connect to several subscriptions at the same time**, for example your GitHub Copilot subscription and your personal Claude subscription. You can then choose which model to use in each mode:

- **Plan** mode: the agent analyzes and proposes changes without modifying files.
- **Build** mode: the agent writes code and runs commands.

For example, you can use a stronger model from one subscription for planning and a model from another subscription for building. This spreads your usage across subscriptions so your credits last longer. Press `Tab` to switch between the two modes, and use `/models` to pick the model for the current mode.

## Installation

Install OpenCode in your home directory by running the following command on the cluster:

```bash
curl -fsSL https://opencode.ai/v2/install | bash
```

Then start it with `opencode` from your project folder.

For other installation methods and documentation, see https://opencode.ai/.

## Connect your Claude subscription

OpenCode uses your Claude subscription through the Claude Code CLI, so **Claude Code must already be installed on the same machine**. If it isn't, follow the [Claude Code installation](#claude-code) section above and log in once by running `claude`.

1. Start OpenCode with `opencode`.
2. Type `/connect` and press `Enter`.
3. Select **Claude Agent** from the list.
4. Follow the on-screen instructions.

When it's done, the Claude models show up in `/models`.

## Connect your GitHub Copilot subscription

1. In OpenCode, type `/connect` again.
2. Select **GitHub Copilot**.
3. Follow the instructions: OpenCode shows a code and a GitHub URL. Open the URL in a browser on your own computer, enter the code, and authorize the app.
4. Return to the terminal. The Copilot models now show up in `/models`.


---

# Notes
- **Do not edit the shared copy in `/mnt/data/unisr-data/agent-rules` in place.** Propose changes on the repository instead; the cluster copy is updated with `git pull`.
- The rules guide the agent's behavior but they are not a technical enforcement: always review what your agent does, especially in autonomous modes.
