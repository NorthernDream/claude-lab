# claude-archive

Snapshots of historical `.claude/` versions.

## Naming convention

One subdirectory per snapshot: `YYYY-MM-DD-<short-tag>/`

For example:
- `2026-04-30-initial/`
- `2026-05-15-v2-with-redis-cluster/`

Each subdirectory holds the full `.claude/` tree (`CLAUDE.md` + `skills/`).

## Archive

```bash
cp -r .claude claude-archive/$(date +%Y-%m-%d)-<tag>
```

## Roll back

```bash
rm -rf .claude
cp -r claude-archive/<dirname> .claude
```
