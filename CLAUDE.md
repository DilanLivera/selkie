# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Selkie is a collection of tools for remoting to servers over a Tailscale network (tailnet). Tools cover SSH, RDP, and other remote access patterns. A tool may be a bash script, an application, or any other form factor appropriate to the task. The runtime environment depends on the individual tool.

## Tools

### lab-connect

Automates SSH-ing into a homelab server over Tailscale and attaching to a Zellij session.

- `lab-connect/lab-connect` — main bash script
- `lab-connect/lab-connect.conf.example` — config template; real config (`lab-connect.conf`) is gitignored
- `lab-connect/REQUIREMENTS.md` — functional and non-functional requirements
- `lab-connect/DESIGN.md` — detailed design including execution flow, component design, and error handling
- `lab-connect/adr/` — architectural decision records (ADR-001 through ADR-004)

**Flow:** load config → check Tailscale auth → SSH with retry → `zellij attach --create` on remote.

**Config lookup:** `$SELKIE_CONFIG` env var → `~/.config/selkie/lab-connect.conf` → `./lab-connect.conf`.

## Conventions

- Each tool lives in its own top-level directory (e.g. `lab-connect/`) containing its source, config templates, requirements, design docs, and ADRs
- Every tool must have requirements, design, and ADR documents before implementation begins
- User-facing messages are prefixed with the tool name, e.g. `[lab-connect]`
- Scripts use `set -euo pipefail`
- Config files are plain key=value sourced by bash, gitignored; `.conf.example` templates are committed
