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

## Workflow Orchestration

1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately – don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

3. Self-Improvement Loop
- After ANY correction from the user: update tasks/lessons.md with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
- If you are can't test the change by running the application, prompt user to test the changes.

5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes – don't over-engineer
- Challenge your own work before presenting it

6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests – then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

### Task Management
1. Plan First: Write plan to tasks/todo.md with checkable items
2. Verify Plan: Check in before starting implementation
3. Implement: Follow TDD and use atomic commits
4. Changelog: Update the CHANGELOG when necessary
5. Track Progress: Mark items complete as you go
6. Explain Changes: High-level summary at each step
7. Document Results: Add review section to tasks/todo.md
8. Capture Lessons: Update tasks/lessons.md after corrections

### Atomic Commits (non-negotiable)
- Each task step that changes code **must end with a commit** before moving to the next step
- Never batch multiple logical changes into one commit, and never defer commits to the end
- The commit is the exit condition for a task step — a step is not done until it is committed

### Core Principles
- Simplicity First: Make every change as simple as possible. Impact minimal code.
- No Laziness: Find root causes. No temporary fixes. Senior developer standards.
- Minimal Impact: Changes should only touch what's necessary. Avoid introducing bugs.

## Commit Convention

```
<type>[optional scope]: <description>
```

- Follow conventional commits
- Each commit represents one logical change (atomic commits)
- Scope is optional but encouraged when the change is confined to a specific area
- Description is lowercase, no trailing period

## GitHub Issues

- Use labels to categorize issues (e.g. `enhancement`, `bug`, `documentation`, `chore`)
- Do not prefix issue titles with type tags — labels handle categorization
