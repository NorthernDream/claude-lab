# claude-archive

存放历史版本的 `.claude/` 完整快照。

## 命名约定

每个快照一个子目录:`YYYY-MM-DD-<简短标签>/`

例如:
- `2026-04-30-initial/`
- `2026-05-15-v2-with-redis-cluster/`

子目录内放整棵 `.claude/` 树(`CLAUDE.md` + `skills/`)。

## 归档操作

```bash
cp -r .claude claude-archive/$(date +%Y-%m-%d)-<标签>
```

## 回滚操作

```bash
rm -rf .claude
cp -r claude-archive/<目录名> .claude
```
