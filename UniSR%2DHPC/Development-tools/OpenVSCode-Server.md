# OpenVSCode Server

Run a standalone, web-based version of VSCode directly on the compute node. This method provides direct access via the node's IP address and is great for low-latency connections within the same network.

*   **Alias**: `coder`
*   **Needs custom image?** Yes — unlike [VSCode Tunnel](/UniSR%2DHPC/Development-tools/VSCode-Tunnel), which works with any Apptainer image, OpenVSCode Server only works on its custom image.

## How to Use

The first run requires you to explicitly launch the executable from its executable path, **on the next runs you can simply use the alias `coder` instead of the full path**.

```bash
/path/to/repo/vscode-server/wrapper.sh <host-workdir> [--bind <host:container> ...]
```

After launching the script, the terminal will display the server's connection details, including the assigned compute node IP and a dynamic port.

## Connecting to the Server

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
