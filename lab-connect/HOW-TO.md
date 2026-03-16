# lab-connect — How-To Guide

## Prerequisites

### Local machine (WSL / Ubuntu)

| Dependency | Purpose |
|---|---|
| [Tailscale](https://tailscale.com/download/linux) | VPN tunnel to your homelab server |
| `ssh` | Included with Ubuntu; used for the remote connection |
| `wslview` | Opens the Tailscale auth URL in your Windows browser. Install via `sudo apt install wslu` |

### Remote server (homelab)

| Dependency | Purpose |
|---|---|
| SSH server | `sshd` must be running and accepting key-based auth |
| [Zellij](https://zellij.dev/documentation/installation) | Terminal multiplexer; lab-connect attaches to a Zellij session on the remote |

Your SSH key must already be set up for passwordless login to the remote server.

---

## Installation

1. Clone the repo into WSL:

   ```bash
   git clone https://github.com/DilanLivera/selkie.git
   ```

2. Symlink the script to a directory on your PATH:

   ```bash
   ln -s "$(pwd)/selkie/lab-connect/lab-connect" ~/.local/bin/lab-connect
   ```

3. Make sure it is executable:

   ```bash
   chmod +x ~/.local/bin/lab-connect
   ```

---

## Configuration

1. Copy the example config to the default config location:

   ```bash
   mkdir -p ~/.config/selkie
   cp selkie/lab-connect/lab-connect.conf.example ~/.config/selkie/lab-connect.conf
   ```

2. Edit the config and fill in your values:

   ```bash
   nano ~/.config/selkie/lab-connect.conf
   ```

### Config keys

| Key | Required | Default | Description |
|---|---|---|---|
| `SSH_USER` | Yes | — | Your username on the remote homelab server |
| `SSH_HOST` | Yes | — | Tailscale IP or hostname of your homelab server |
| `SSH_RETRY_COUNT` | No | `3` | Number of SSH connection attempts before giving up |
| `SSH_RETRY_DELAY` | No | `5` | Seconds to wait between retry attempts |

### Config lookup order

The script looks for a config file in this order and uses the first one found:

1. Path set by the `SELKIE_CONFIG` environment variable
2. `~/.config/selkie/lab-connect.conf`
3. `./lab-connect.conf` (current directory — useful during development)

---

## Usage

From a WSL shell:

```bash
lab-connect
```

From PowerShell or Windows Terminal:

```powershell
wsl lab-connect
```

That's it. The script handles everything else automatically.

---

## What happens

When you run `lab-connect`, it performs the following steps:

1. **Loads config** — reads your SSH and retry settings
2. **Checks Tailscale auth** — if Tailscale needs re-authentication, it opens the login URL in your Windows browser and waits up to 2 minutes for you to complete Google SSO
3. **Connects via SSH with retry** — attempts to SSH into your homelab server, retrying on failure up to `SSH_RETRY_COUNT` times with a `SSH_RETRY_DELAY` second pause between attempts
4. **Attaches to Zellij** — on the remote server, attaches to the most recent Zellij session if one exists, or creates a new session if none exist

When the Zellij session ends (or you detach), the SSH connection closes and you are back at your local terminal.

---

## Troubleshooting

| Error message | Cause | Fix |
|---|---|---|
| `[lab-connect] tailscale not found. Is Tailscale installed in WSL?` | Tailscale CLI is not installed in WSL | [Install Tailscale for Linux](https://tailscale.com/download/linux) inside your WSL distro |
| `[lab-connect] Config file not found...` | No config file found in any of the lookup locations | Copy `lab-connect.conf.example` to `~/.config/selkie/lab-connect.conf` and fill in your values |
| `[lab-connect] SSH_USER is not set in config.` | Config file exists but `SSH_USER` is blank | Edit your config file and set `SSH_USER` to your remote username |
| `[lab-connect] SSH_HOST is not set in config.` | Config file exists but `SSH_HOST` is blank | Edit your config file and set `SSH_HOST` to your server's Tailscale IP or hostname |
| `[lab-connect] Tailscale auth required, opening browser...` | Tailscale session has expired | Complete the Google SSO login in the browser window that opens |
| `[lab-connect] Tailscale auth timed out...` | SSO was not completed within 2 minutes | Run `tailscale login` manually and complete auth, then re-run `lab-connect` |
| `[lab-connect] Could not extract auth URL...` | `tailscale login` did not output a URL | Run `tailscale login` manually to diagnose the issue |
| `[lab-connect] SSH failed after N attempts...` | All SSH retry attempts failed | Check that Tailscale is connected (`tailscale status`), that the remote server is online, and that your SSH key is set up correctly |
