# ADR-003: Configurable SSH Retry Logic

**Date:** 2026-03-12
**Status:** Accepted

---

## Context

SSH connections over Tailscale can fail transiently (e.g. Tailscale not yet ready, network instability on mobile hotspot). A single failure should not require the user to re-run the command manually.

## Decision

Implement a retry loop around the SSH connection attempt with:
- Default retry count: **3**
- Retry count configurable via a config file or environment variable
- Clear error output after all retries are exhausted

## Reasoning

- The user explicitly requested 3 retries with configurability
- Transient failures are expected given the mobile hotspot + Tailscale setup
- Configurable retries avoid hardcoding and allow tuning without modifying the script

## Consequences

- A config file or environment variable mechanism must be established (shared with FR-7)
- Retry delay between attempts should be decided at implementation time (not specified in requirements)
