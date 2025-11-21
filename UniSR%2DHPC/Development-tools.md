# Development Environment Launchers

This guide provides a unified overview of a suite of scripts that launch common development environments—**VSCode Tunnel**, **OpenVSCode Server**, and **JupyterLab**—inside an **Apptainer** container within an HPC environment using **SLURM**.

These tools provide a convenient wrapper to get a fully functional development environment running on a compute node, accessible from your local machine.

> You are highly encouraged to check the official github repository of the full collection of these tools that can be found [here](https://github.com/AI-UniSR/cluster-entrypoints) along with custom documentation files for each tool implemented. You can find a stable version of the repository in the cluster at `/mnt/data/unisr-data/entrypoints`

---


[[_TOC_]]

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

1.  **First-Time Setup**: On the first run, the script will perform a brief setup and add a unique alias to your `~/.bashrc` file. This means you only need to use the full script path once; afterward, you can use the simple alias (e.g., `code-tunnel`, `coder`, or `jupyter`).
2.  **`wrapper.sh`**: This is the entry point. It validates your input arguments, parses any extra bind mounts, and launches an interactive job on a compute node using SLURM (`srun`).
3.  **`job.sh`**: This script runs on the compute node. It prepares persistent directories for user data, determines the node's IP address if needed, and finally executes the appropriate service (VSCode, Jupyter, etc.) inside an Apptainer container with your specified directories mounted.

## Common Usage
First you either need to clone the [original repository](https://github.com/AI-UniSR/cluster-entrypoints) to your home directory or access a stable version at `/mnt/data/unisr-data/entrypoints`. From there, locate the tool you want to use and `cd` into its dedicated directory.

All three tools are launched using a similar command format:

```bash
<tool-executable> <host-workdir> [--bind <host:container> ...]
```

### Parameters

*   `<host-workdir>` — **Required**.
    The directory on the host machine that you want to mount as your main project folder inside the container.
*   `--bind <host:container>` — **Optional**.
    Use this to specify additional directories to mount inside the container. This argument is passed directly to Apptainer, and you can use it multiple times.

### Example

This example mounts a project directory and two additional directories for datasets and tools. This command format works for all the tools described below.

```bash
./code-server/wrapper.sh ~/projects/mycode \
   --bind /mnt/data/datasets:/workspace/datasets \
   --bind /mnt/tools:/workspace/tools
```

---

# Tools

Below are the specific instructions for launching and connecting to each development environment.

## VSCode Tunnel

Connect to a remote machine via a secure tunnel without needing to open SSH ports. This method is ideal for accessing your environment from anywhere through a `vscode.dev` URL.

*   **Alias**: `code-tunnel`

### How to Use
The first run requires you to explicitly launch the executable from its executable path, **on the next runs you can simply use the alias `code-tunnel` instead of the full path**.

```bash
/path/to/repo/vscode-tunnel/src/launch.py <host-workdir> [--bind <host:container> ...]
```
Additionally, the first run will require you to log in with a GitHub or Microsoft account to authenticate the tunnel. This is a one-time operation, as your login will be persisted for future sessions.

After authentication, the service will provide a URL:

```
Open this link in your browser https://vscode.dev/tunnel/your-tunnel-name
```

### Connecting to the Tunnel

You have two options to access your environment:

- **From the Browser**
    1. Simply `Ctrl+Click` the link printed in your terminal.
    1. A full-featured VSCode editor will open in your web browser, connected to your HPC environment.

- **From a Local VSCode Application (Recommended)**

  1.  **Install the "Remote Tunnels" Extension**:
      -   Open your local VSCode application.
      -   Go to the Extensions view (`Ctrl+Shift+X`).
      -   Search for `Remote Tunnels` (published by Microsoft) and install it.

  2.  **Connect to the Tunnel**:
      -   Open the Command Palette (`Ctrl+Shift+P`) and type `Remote-Tunnels: Connect to Tunnel...`.
      -   Log in with the same GitHub/Microsoft account used on the remote machine.
      -   Select your tunnel from the list and click to connect.
      -   A new VSCode window will open, connected to your remote project. Click "Open Folder" to browse and open your `<host-workdir>`.

**This is only a brief description of the tool, please check the [official guide](https://github.com/AI-UniSR/cluster-entrypoints/blob/main/vscode-tunnel/vscode-tunnel.md) for more in depth details on how to use the latest version.**

---

## OpenVSCode Server

Run a standalone, web-based version of VSCode directly on the compute node. This method provides direct access via the node's IP address and is great for low-latency connections within the same network.

*   **Alias**: `coder`

### How to Use
The first run requires you to explicitly launch the executable from its executable path, **on the next runs you can simply use the alias `coder` instead of the full path**.

```bash
/path/to/repo/vscode-server/wrapper.sh <host-workdir> [--bind <host:container> ...]
```

After launching the script, the terminal will display the server's connection details, including the assigned compute node IP and a dynamic port.

### Connecting to the Server

The output will include a direct link with an access token.

**Example Output:**

```
🤖 Starting OpenVSCode Server:
Base image     -> /path/to/image.sif
User data      -> /home/user/coder
Dev folder     -> /home/user/projects/mycode
On             -> 172.21.203.43:50012
```

```
🌐 Access OpenVSCode Server:
http://172.21.203.43:50012/?tkn=vscode-123456789
```

1.  Copy the full URL.
2.  Paste it into your web browser to start coding.

**This is only a brief description of the tool, please check the [official guide](https://github.com/AI-UniSR/cluster-entrypoints/blob/main/vscode-server/vscode-server.md) for more in depth details on how to use the latest version.**

---

## JupyterLab Server

Run a classic JupyterLab server on the compute node. This is perfect for data science, notebooks, and interactive computing.

*   **Alias**: `jupyter`

### How to Use
Similar to the OpenVSCode Server, this script starts a web service on the compute node and provides a direct access link.
The first run requires you to explicitly launch the executable from its executable path, **on the next runs you can simply use the alias `jupyter` instead of the full path**.

```bash
/path/to/repo/juoyter-server/wrapper.sh <host-workdir> [--bind <host:container> ...]
```

### Connecting to the Server

The terminal will print the connection details and a URL containing the required access token.

**Example Output:**

```
🤖 Starting JupyterLab Server:
Base image     -> /path/to/image.sif
User data      -> /home/user/coder
Dev folder     -> /home/user/projects/mycode
On             -> 172.21.203.43:50012
```

```
🌐 Access JupyterLab Server:
http://172.21.203.43:50012/?tkn=vscode-123456789
```

1.  Copy the first URL provided in the output.
2.  Paste it into your web browser to access the JupyterLab interface.


**This is only a brief description of the tool, please check the [official guide](https://github.com/AI-UniSR/cluster-entrypoints/blob/main/jupyter-server/jupyter-server.md) for more in depth details on how to use the latest version.**

---
## 🧩 General Notes

*   The main development directory (`<host-workdir>`) must exist before running the script.
*   User configurations, extensions, and data are persisted in a hidden directory in your home (e.g., `~/.vscode-tunnel` or `~/coder`). Do not delete these if you wish to maintain your settings across sessions.
*  [VSCode Tunnel](#vscode-tunnel) works on any Apptainer image. However, currently, [OpenVSCode server](#openvscode-server) and [JupyterLab server](#jupyterlab-server) will only work on their custom image.
---
