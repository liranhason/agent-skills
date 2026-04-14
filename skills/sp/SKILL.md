---
name: sp
description: "Shortcut alias for /superplan — create and manage incremental implementation plans. Subcommands: create (default), status, next, review."
type: meta
allowed-tools: Bash(git:*, ls:*, cat:*), Read, Glob, Grep, Edit, Write, Agent, Task
argument-hint: <description or ticket_id> | status | next [plan_file] [--yes] | review [plan_file]
---

This is a shortcut alias for `/superplan`. Read and follow the instructions in the `superplan` skill (installed alongside this one as `~/.claude/skills/superplan/SKILL.md`).

Execute exactly as if the user typed `/superplan` with the same arguments — same routing, same subcommands, same behavior.
