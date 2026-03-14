# ADR-004: Tailscale Browser-Based Auth Handling

**Date:** 2026-03-12
**Status:** Accepted

---

## Context

Tailscale requires periodic re-authorization via a browser-based Google SSO flow. This currently blocks the SSH workflow and requires manual intervention. Options considered:

1. Ignore it — let SSH fail and tell the user to auth manually
2. Detect auth requirement and prompt the user to open a browser
3. Detect auth requirement, automatically open the browser, and wait for completion

## Decision

Detect when Tailscale requires re-authorization, automatically open the browser to the Tailscale auth URL, and wait for the user to complete Google SSO before proceeding.

## Reasoning

- The user experiences this on every SSH attempt, so it must be handled in-flow
- Automatically opening the browser reduces friction to a single action (completing the Google SSO login)
- Waiting for auth completion before retrying SSH keeps the flow sequential and predictable
- The user is comfortable with browser-based SSO as the auth mechanism

## Consequences

- The script must be able to detect Tailscale auth state (e.g. by running `tailscale status` or catching SSH errors)
- Opening a browser from WSL requires `wslview` or `explorer.exe` — this is a known WSL pattern
- If the Tailscale session duration is extended in future, this step will naturally become less frequent (OQ-1)
