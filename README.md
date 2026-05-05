# claude-lab

*[English](README.md) | [中文](README.zh.md)*

Experimental sandbox for prototyping Claude Code configuration — skills, hooks, permissions, agents, settings, workflows. Test changes here in isolation, verify, then promote them into the global `~/.claude/` setup.

## Why

Editing `~/.claude/` directly is risky:
- A bad permission can break the next session
- A bad skill silently gets picked up on next invocation
- No rollback if you can't remember what you changed

This project flips it around: every change starts inside `claude-lab/.claude/`, gets verified by the user, then is written through to global. If something breaks, `claude-archive/` holds a directory of dated snapshots to roll back from.

## Layout

```
claude-lab/
├── .claude/
│   ├── CLAUDE.md           # project workflow contract — start here
│   ├── CLAUDE.zh.md        # Chinese mirror
│   ├── settings.local.json # project-only permissions (gitignored)
│   └── skills/             # checked-out skills during in-flight modifications
├── claude-archive/         # dated rollback snapshots (contents gitignored)
├── README.md               # project intro (you're here)
├── README.zh.md            # Chinese mirror
└── .gitignore
```

## Start here

Read `.claude/CLAUDE.md`. It's the workflow contract — describes how to safely modify global Claude Code config by staging through this project, with bilingual maintenance, in-flight tracker, and archiving rules.

## Bilingual convention

Agent-facing markdown files in this project keep an English source plus a `.zh.md` Chinese mirror in sync. English is canonical.
