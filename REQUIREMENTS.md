# Selkie — Requirements Document

## 1. Overview

**Project:** selkie
**Purpose:** Tooling to reduce the multi-step process of SSH-ing into a homelab Linux server and accessing Claude CLI into a single command.

---

## 2. Problem Statement

Accessing Claude CLI on a remote homelab server currently requires multiple manual steps:

1. Manually connect Windows machine to mobile hotspot WiFi
2. Run SSH command
3. Complete Tailscale browser-based authorization (Google SSO)
4. Start Zellij
5. Run the Claude CLI command

The goal is to collapse steps 2–5 into a single command that can be invoked from Windows Terminal.

> **Note:** Step 1 (switching WiFi to mobile hotspot) is a manual hardware/OS action and is out of scope. The hotspot is assumed to always be on and available.

---

## 3. Environments

### 3.1 Local (Client) Machine
| Property | Value |
|---|---|
| OS | Windows |
| Terminal | Windows Terminal |
| Shell | PowerShell (primary), WSL available |
| WSL Distro | Ubuntu |

### 3.2 Remote (Homelab) Machine
| Property | Value |
|---|---|
| OS | Linux |
| Network | Tailscale (tailnet) |
| SSH command | `ssh user@tailnet_ip` |
| Auth | SSH key (no password) |
| Session manager | Zellij |
| Target application | Claude CLI |

---

## 4. Functional Requirements

### FR-1: Single Command Invocation
The tool must be invocable as a single short command (e.g. `selkie`) from Windows Terminal without additional arguments required for normal use.

### FR-2: Scripting Environment
The script must run inside WSL (Ubuntu) and be available on the WSL PATH so it can be called from Windows Terminal.

### FR-3: Tailscale Authorization
The tool must handle Tailscale authentication. When Tailscale requires re-authorization, it opens a browser page requiring Google SSO login. The tool must:
- Detect when Tailscale auth is required
- Open the browser authorization page
- Wait for the user to complete Google SSO
- Proceed with SSH once authorized

### FR-4: SSH Connection
The tool must establish an SSH connection to the homelab server using the existing SSH key.

### FR-5: Zellij Session Management
Upon successful SSH, the tool must:
- Attach to the last active Zellij session if one exists
- Create a new Zellij session (with the standard layout) if no session exists

### FR-6: Retry Logic
If the SSH connection fails, the tool must:
- Retry automatically up to **3 times** (default, configurable)
- Print a clear error message after all retries are exhausted

### FR-7: Configuration
The following values must be configurable (not hardcoded):
- SSH user and host (tailnet IP/hostname)
- Number of SSH retry attempts (default: 3)

---

## 5. Non-Functional Requirements

### NFR-1: Simplicity
The tool should be as simple as possible. No unnecessary dependencies.

### NFR-2: Credential Storage
The user is comfortable storing SSH keys and configuration values in script/config files on the local machine (WSL).

### NFR-3: Error Messaging
Errors should be human-readable and actionable (e.g. "SSH failed after 3 attempts — check your Tailscale connection").

---

## 6. Out of Scope

- Automating WiFi/hotspot switching on Windows
- Managing or configuring Tailscale itself
- Starting or managing the Claude CLI command (user will run it manually inside Zellij)
- Windows-native PowerShell scripting (WSL is the chosen environment)

---

## 7. Success Criteria

The project is successful when:
- The user can open Windows Terminal and run a single command
- The command handles Tailscale auth, SSH, and Zellij session attachment automatically
- On failure, the tool retries and provides a clear error message
- The end state is the user inside their Zellij session, ready to run Claude

---

## 8. Open Questions

| # | Question | Status |
|---|---|---|
| OQ-1 | How long does a Tailscale session last before re-auth is required? | Unresolved |
| OQ-2 | Should `selkie` be callable directly from PowerShell (via `wsl selkie`) or does the user always switch to a WSL shell first? | Unresolved |
| OQ-3 | What is the exact Zellij layout used? Does it need to be version-controlled in this repo? | Unresolved |
