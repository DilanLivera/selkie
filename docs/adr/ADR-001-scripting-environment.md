# ADR-001: Use WSL (Ubuntu) as the Scripting Environment

**Date:** 2026-03-12
**Status:** Accepted

---

## Context

The user's local machine is Windows, running Windows Terminal with PowerShell. The goal is to invoke a script that SSH-es into a Linux homelab server. Scripting options considered:

- PowerShell (native Windows)
- WSL (Ubuntu) — already installed on the machine
- A cross-platform tool (e.g. Python)

## Decision

Use WSL (Ubuntu) as the scripting environment. The script will be a shell script placed on the WSL PATH.

## Reasoning

- The target server is Linux; bash scripting is idiomatic for SSH, Zellij, and CLI tooling on Linux
- WSL is already installed — no new tooling required
- SSH key management and `ssh` CLI are native and simpler in a Unix shell than PowerShell
- The user expressed no preference for script location and is comfortable with WSL
- Avoids PowerShell complexity around SSH, process management, and Unix-style tooling

## Consequences

- The user must launch the script from a WSL shell (or via `wsl selkie` from PowerShell)
- Windows-native features (e.g. native WiFi switching) are not easily accessible from WSL
