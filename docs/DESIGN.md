# Selkie — Design Document

**Date:** 2026-03-13
**Status:** Draft
**References:** [REQUIREMENTS.md](../REQUIREMENTS.md), ADR-001 through ADR-004

---

## 1. Overview

Selkie is a single bash script that runs inside WSL (Ubuntu) and automates the full workflow of connecting to a homelab server via SSH over Tailscale, and attaching to a Zellij session. It is invoked as a single command from Windows Terminal.

---

## 2. Repository Structure

```
selkie/
├── lab-connect          # Main entry point script
├── lab-connect.conf     # User configuration file
├── docs/
│   ├── DESIGN.md        # This document
│   ├── adr/
│   │   ├── ADR-001-scripting-environment.md
│   │   ├── ADR-002-zellij-session-management.md
│   │   ├── ADR-003-retry-logic.md
│   │   └── ADR-004-tailscale-auth.md
└── REQUIREMENTS.md
```

---

## 3. Installation

1. Clone the repo into WSL (Ubuntu)
2. Copy or symlink `lab-connect` to a directory on the WSL `$PATH` (e.g. `~/.local/bin/lab-connect`)
3. Make it executable: `chmod +x ~/.local/bin/lab-connect`
4. Edit `lab-connect.conf` with SSH user, host, and any overrides

The script can then be invoked:
- From a WSL shell: `lab-connect`
- From PowerShell/Windows Terminal: `wsl lab-connect`

---

## 4. Configuration

Configuration is stored in `lab-connect.conf` as key=value pairs, sourced by the script at startup.

```ini
# lab-connect.conf

SSH_USER=your_username
SSH_HOST=your.tailnet.hostname.or.ip
SSH_RETRY_COUNT=3
SSH_RETRY_DELAY=5
```

| Key | Default | Description |
|---|---|---|
| `SSH_USER` | _(required)_ | Username on the remote server |
| `SSH_HOST` | _(required)_ | Tailnet IP or hostname of the homelab server |
| `SSH_RETRY_COUNT` | `3` | Number of SSH connection attempts before giving up |
| `SSH_RETRY_DELAY` | `5` | Seconds to wait between retry attempts |

The config file is looked up in the following order:
1. Path provided by `SELKIE_CONFIG` environment variable
2. `~/.config/selkie/lab-connect.conf`
3. `./lab-connect.conf` (repo directory, for development)

---

## 5. Architecture

### 5.1 Execution Flow

```
lab-connect
  │
  ├─ 1. Load config
  │
  ├─ 2. Check Tailscale auth
  │       ├─ [needs auth] → open browser → wait for auth → continue
  │       └─ [ok] → continue
  │
  ├─ 3. SSH with retry loop
  │       ├─ attempt 1..N
  │       │     ├─ [success] → remote session begins
  │       │     └─ [fail] → wait SSH_RETRY_DELAY → retry
  │       └─ [all failed] → print error and exit 1
  │
  └─ 4. On remote: attach or create Zellij session
          ├─ [session exists] → zellij attach (most recent)
          └─ [no session] → zellij (new session, default layout)
```

### 5.2 Components

#### `lab-connect` — Main Script

Single bash script. No external dependencies beyond standard Unix tools and the software already present on the local WSL machine (`tailscale`, `ssh`, `wslview`).

Responsibilities:
- Source config
- Run Tailscale auth check
- Run SSH with retry loop
- Pass the Zellij attach-or-create command to the remote shell via SSH

#### `lab-connect.conf` — Configuration File

Plain key=value file. Not committed to the repo (added to `.gitignore`). A `lab-connect.conf.example` template is committed instead.

---

## 6. Detailed Component Design

### 6.1 Tailscale Auth Check

Tailscale auth state is checked by running `tailscale status` on the local WSL machine before attempting SSH.

**Detection logic:**
- Run `tailscale status`
- If output contains `NeedsLogin` or the command exits non-zero → auth required
- Run `tailscale login` which outputs a URL
- Extract the URL and open it with `wslview <url>` (opens the default Windows browser from WSL)
- Poll `tailscale status` until auth is confirmed (up to a timeout)

```
tailscale status
  ├─ ok           → proceed to SSH
  └─ NeedsLogin   → tailscale login → extract URL → wslview URL
                    → poll until status ok → proceed to SSH
```

### 6.2 SSH with Retry

```bash
for attempt in $(seq 1 $SSH_RETRY_COUNT); do
    ssh -o ConnectTimeout=10 $SSH_USER@$SSH_HOST <remote_command>
    if [ $? -eq 0 ]; then exit 0; fi
    echo "[lab-connect] SSH attempt $attempt/$SSH_RETRY_COUNT failed. Retrying in ${SSH_RETRY_DELAY}s..."
    sleep $SSH_RETRY_DELAY
done
echo "[lab-connect] SSH failed after $SSH_RETRY_COUNT attempts. Check your Tailscale connection."
exit 1
```

### 6.3 Zellij Session Management (Remote)

The Zellij command is executed on the remote server via SSH. The remote command passed through SSH:

```bash
zellij attach --create
```

`zellij attach --create` attaches to the most recently used session if one exists, or creates a new session if none exist. This satisfies FR-5 with a single command and no additional logic required on the local side.

The full SSH invocation:

```bash
ssh -t $SSH_USER@$SSH_HOST "zellij attach --create"
```

The `-t` flag allocates a pseudo-TTY, which is required for interactive terminal applications like Zellij.

---

## 7. Error Handling

| Scenario | Behaviour |
|---|---|
| Tailscale not installed on WSL | Print error: "tailscale not found. Is Tailscale installed in WSL?" and exit |
| Tailscale auth times out (user doesn't complete SSO) | Print error: "Tailscale auth timed out. Run `tailscale login` manually." and exit |
| SSH fails all retries | Print error: "SSH failed after N attempts — check your Tailscale connection." and exit 1 |
| SSH succeeds but Zellij not found on remote | SSH error will propagate naturally; no special handling required |

---

## 8. Security Considerations

- `lab-connect.conf` is excluded from version control via `.gitignore` to prevent accidental credential leakage
- SSH authentication uses existing SSH keys — no passwords or tokens are stored in the script
- No Tailscale tokens or API keys are stored; auth is delegated to the `tailscale` CLI and browser SSO

---

## 9. Resolved Open Questions

| # | Question | Resolution |
|---|---|---|
| OQ-2 | Should `lab-connect` be callable from PowerShell directly? | User can call via `wsl lab-connect` from PowerShell, or open a WSL shell first. Both work without extra scripting. |
| OQ-3 | Does the Zellij layout need to be versioned? | `zellij attach --create` uses Zellij's default behaviour. Layout versioning deferred unless user specifies a custom layout. |

| # | Question | Status |
|---|---|---|
| OQ-1 | How long does a Tailscale session last before re-auth is required? | Still unresolved — does not block implementation; auth check handles it regardless of duration. |
