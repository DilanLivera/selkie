# ADR-002: Attach to Last Zellij Session or Create New

**Date:** 2026-03-12
**Status:** Accepted

---

## Context

Upon SSH-ing into the homelab, the user wants to land inside Zellij. The options are:

1. Always create a new Zellij session
2. Always attach to a named session
3. Attach to the last active session if one exists, otherwise create a new one

The user works with a consistent Zellij layout and wants to resume where they left off when a session is already running.

## Decision

Use `zellij attach --create` (or equivalent) to attach to the most recent session if one exists, and create a new session with the standard layout if none exists.

## Reasoning

- The user explicitly stated they want to connect to the last opened session
- This avoids accumulating orphaned sessions while still providing a fallback for fresh starts
- Zellij supports session listing and attachment natively

## Consequences

- If multiple sessions exist, behaviour depends on Zellij's definition of "last" — this may need clarification during implementation
- The standard layout file may need to be version-controlled in this repo (Open Question OQ-3)
