# Superset

Superset is a desktop application for orchestrating multiple CLI coding agents running in parallel across isolated git worktrees. Rather than being an agent framework itself, it is a management layer that sits on top of existing CLI agents (Claude Code, Codex, etc.) and handles the operational complexity of running many of them simultaneously.

## What Problem It Solves

Sequential CLI agents are a bottleneck: you submit a task, wait for the agent to finish, then submit the next. Superset treats coding agent invocations as parallel workers — you issue many tasks at once, each agent runs in its own isolated environment, and a unified UI lets you monitor status and review output across all of them.

The specific pain point it addresses is context-switching overhead when managing multiple in-flight agents: instead of juggling separate terminal windows or tmux sessions, Superset provides a single interface for status monitoring, notifications, diff review, and PR management. Users in the HN launch thread reported managing 40–50 concurrent agent sessions across multiple repositories without losing track.

## Architecture

The core isolation primitive is the git worktree. Each task gets its own branch and working directory, so agents cannot interfere with each other and results can be reviewed independently before merging.

Built on Electron + React, with Bun, Turborepo, Vite, Drizzle ORM, Neon, and tRPC. The application manages terminal sessions within worktrees and provides real-time status tracking. A Remote Workspaces feature allows agents to run on remote machines while being monitored from the local desktop.

Agents are invoked as CLI processes, making Superset agent-agnostic — any CLI agent can be used.

## Current State (May 2026)

- YC P26 company; three-person team (Avi Peltz, Kiet, Satya)
- ~11k GitHub stars, 161 releases, active development
- Desktop app available; Remote Workspaces recently shipped
- Licensed under Elastic License 2.0 (source-available, not OSI open source)
- Free tier available; team/cloud features planned as paid tier

## Limitations and Tradeoffs

- Desktop Electron app: resource-intensive (~2GB memory reported by some users); some users have reported freezing and rendering problems at scale
- Requires account login
- Model selection delegated to the underlying CLI agent, not configurable from within Superset
- Competitive with similar tools (Conductor and others); differentiates on terminal-first design and remote workspace support

## See Also

- [Multi-Agent Coordination](../concepts/multi-agent.md) — the parallel workers pattern Superset operationalizes
- [Pi](pi.md) — another CLI-layer coding agent tool, with a different philosophy (minimal harness, token efficiency)
- [Claude Code](claude-code.md) — one of the CLI agents Superset is designed to orchestrate
