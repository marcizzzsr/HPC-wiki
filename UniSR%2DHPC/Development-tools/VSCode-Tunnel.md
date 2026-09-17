# VSCode Tunnel

Connect to a remote machine via a secure tunnel without needing to open SSH ports. This method is ideal for accessing your environment from anywhere through a `vscode.dev` URL, and it works with **any** Apptainer image (no custom image required).

*   **Alias**: `code-tunnel`

## What is a VS Code Tunnel?

VS Code Tunnels let you securely connect to a remote machine — in this case a cluster compute node — from any other device, without configuring SSH, VPNs, or opening firewall ports. A tunnel is started on the "host" (the compute node), and Microsoft's tunneling service securely routes the connection, letting you access that host's file system, terminal, and compute power directly through your local VS Code app or a web browser.

This is what makes it possible to code from a lightweight laptop or tablet while the actual work — running, compiling, and accessing files — happens on the cluster.

## How `code-tunnel` Works Under the Hood

For technical reasons, the tunnel implementation on the cluster works a bit differently than a standard VS Code tunnel. The `code-tunnel` command is essentially a **wrapper** around a chain of calls: it requests a node from SLURM, creates an Apptainer container from a specified image, binds the default and requested volumes, and starts the VSCode Tunnel service inside that container.

1.  **Reads config** — Parses arguments from the CLI or from a configuration file stored in the working directory (or falls back to a default one). CLI arguments override the base config.
2.  **SLURM** — Allocates an interactive compute node with the requested resources.
3.  **Apptainer** — Starts the container using the specified image, binding your workdir and any additional requested binds.
4.  **Service** — Executes the VSCode Tunnel service **inside the container**.

In short: `code-tunnel ~/projectX` connects to the login node over SSH, which submits a SLURM job to an available compute node; on that node, Apptainer starts your container, and the VSCode Server inside it is what your local VSCode Client (or vscode.dev in the browser) actually tunnels into.

## The Configuration File

Create a file named `.code-tunnel-config.yaml` in your working directory to declare the SLURM resources and Apptainer image/binds for your session. Example:

```yaml
# VSCode Tunnel Configuration

slurm:
  partition: "interactive"
  time: "8:00:00"
  cpus: 4
  memory: "16G"
  nodes: 1

apptainer:
  image_path: "/mnt/data/unisr-data/apptainer_images/images/monai-detection/image.sif"
  overlays:
    code: "/mnt/data/unisr-data/apptainer-image-registry/overlays/vscode.sqsh:ro"
    writable: # writable overlay to mimic docker behavior
      enabled: false
      name: "writable-overlay"
      auto: true # create automatically if missing
      sparse: true # use sparse allocation
      size: 1024 # max size in MB

binds:
  - "/mnt/data/unisr-data/datasets:/data/slow/"
  - "/mnt/beegfs/nvme/datasets:/data/fast/"
  - "/mnt/data/unisr-data/models:/models"

# GPU and tunnel configuration
gpu:
  enabled: false
tunnel:
  name: "monai-detection"
```

A quick field-by-field overview:

*   `slurm` — the resources requested for the interactive job (`partition`, `time`, `cpus`, `memory`, `nodes`).
*   `apptainer.image_path` — the `.sif` image used to start the container.
*   `apptainer.overlays.code` — a read-only overlay providing the VSCode/tooling layer on top of the base image.
*   `apptainer.overlays.writable` — an optional writable overlay to mimic Docker-like persistent writes inside the container (`enabled`, `name`, `auto`-create if missing, `sparse` allocation, max `size` in MB).
*   `binds` — extra host↔container directory mounts, in addition to your workdir.
*   `gpu.enabled` — whether to request GPU resources for the job.
*   `tunnel.name` — the name your tunnel will be registered under (defaults to `unisr-code-tunnel` if omitted).

## Getting Connected: Step by Step

1.  **Create the configuration file** — add `.code-tunnel-config.yaml` to your working directory with your requested SLURM resources and Apptainer image, as shown above.

2. **Install the tool** (skip if not first run) - install with:
    ```bash
    sh /mnt/data/unisr-data/entrypoints/vscode-tunnel/install.sh
    ```
2.  **Start the tunnel** — launch it with:

    ```bash
    code-tunnel <host-workdir> [--bind <host:container> ...]
    ```

    On first use you'll be asked to authenticate with a GitHub or Microsoft account — this is a one-time operation, your login is persisted for future sessions. After authentication, the service prints a URL:

    ```
    Open this link in your browser https://vscode.dev/tunnel/your-tunnel-name
    ```
3.  **Open the tunnel** — you have two options:

    - **From the Browser**: `Ctrl+Click` the printed link to open a full-featured VSCode editor in your web browser, connected to your HPC environment.
    - **From a Local VSCode Application (Recommended)**:
        1.  Install the **Remote Tunnels** extension (published by Microsoft) in your local VSCode via the Extensions view (`Ctrl+Shift+X`).
        2.  Open the Command Palette (`Ctrl+Shift+P`) and run `Remote-Tunnels: Connect to Tunnel...`.
        3.  Log in with the same GitHub/Microsoft account used on the remote machine.
        4.  Select your tunnel from the list — the default name is `unisr-code-tunnel` unless overridden in the config.
        5.  A new VSCode window opens, connected to your remote session. Click "Open Folder" to browse and open your `<host-workdir>`.

**This is only a brief description of the tool, please check the [official guide](https://github.com/AI-UniSR/cluster-entrypoints/blob/main/vscode-tunnel/vscode-tunnel.md) for more in depth details on how to use the latest version.**
