# Connecting to the cluster via SSH


<div style="background-color: #e8f1fa; border-left: 6px solid #1565c0; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #1565c0;">ℹ️ Info</strong>
  <p style="margin: 8px 0 0;">
  In order to connect to our cluster you need to be granted access from our IT Team first. Please, reach out to them in order to configure your account.
  </p>
</div>


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

## Tip: Avoid retyping your password with SSH connection sharing (ControlMaster)

SSH key-pair authentication is not allowed on the cluster, so by default you have to type your password for every new connection (new terminal, `scp`, `rsync`, VSCode, etc.). You can avoid this with SSH **connection multiplexing**: the first connection authenticates with your password and stays open in the background, and any later connection to the same host reuses it without asking for the password again.

Add these lines to your Host entry in `~/.ssh/config`:

```bash
Host hsr-server
    HostName 10.64.79.72
    User surname.name@hsr.it
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 8h
```

- `ControlMaster auto`: reuses an existing shared connection if there is one, otherwise creates a new one.
- `ControlPath`: where the socket file for the shared connection is stored (`%r` = remote user, `%h` = host, `%p` = port).
- `ControlPersist 8h`: keeps the shared connection alive in the background for 8 hours after the last session is closed. Adjust it to your needs.

Now you only type your password on the first `ssh hsr-server`. Any further `ssh`, `scp` or `rsync` to `hsr-server` within the persist window will connect immediately.

Useful commands to manage the shared connection:

```bash
ssh -O check hsr-server   # check whether a shared connection is active
ssh -O exit hsr-server    # close the shared connection
```

<div style="background-color: #fff8e1; border-left: 6px solid #f9a825; padding: 12px; margin: 16px 0; border-radius: 4px;">
  <strong style="color: #f9a825;">⚠️ Note</strong>
  <p style="margin: 8px 0 0;">
  Connection multiplexing is supported on Linux, macOS and WSL. The native Windows OpenSSH client does not support <code>ControlMaster</code>, so on Windows use it from WSL. Also, if the VPN drops, the shared connection may hang: run <code>ssh -O exit hsr-server</code> (or delete the <code>~/.ssh/cm-*</code> socket file) and reconnect.
  </p>
</div>

---

## Optional: Set correct permissions (Linux/macOS)

Make sure your SSH config file is readable only by you:

```bash
chmod 600 ~/.ssh/config
```

---

This setup saves you from typing the full SSH command every time!