# Connecting to the cluster via SSH

To connect to our cluster, first establish a suitable VPN connection, currently the ones supported are:
- HSR VPN 
- S-RACE Azure VPN P2S

Use the following SSH command in your terminal or command prompt:

```bash
ssh surname.name@hsr.it@10.64.79.72
```

Replace `surname.name@hsr.it` with your actual account email.

---

## Setting Up SSH Config for Easier Access

Typing the full command every time can be cumbersome. You can simplify this by configuring your SSH client.

### Step 1: Locate or create your SSH config file

- **Linux/macOS**: The config file is at `~/.ssh/config`
- **Windows** (using OpenSSH client, e.g. in PowerShell or WSL): The config file is at `%USERPROFILE%\.ssh\config`

If the file doesn’t exist, create it.

### Step 2: Add a Host entry

Open the config file with a text editor and add the following, replacing `surname.name@hsr.it` with your account:

```bash
Host hsr-server
    HostName 10.64.79.72
    User surname.name@hsr.it
```
You can choose the host config name to be whatever you want, here we used `hsr-server` as an example.

### Step 3: Save and close the file

### Step 4: Connect using the shortcut

Now you can connect simply by running:

```bash
ssh hsr-server
```

---

## Optional: Set correct permissions (Linux/macOS)

Make sure your SSH config file is readable only by you:

```bash
chmod 600 ~/.ssh/config
```

---

This setup saves you from typing the full SSH command every time!