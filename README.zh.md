# claude-lab

Claude Code 配置的实验沙盒 —— skills、hooks、permissions、agents、settings、workflows。在这里隔离测试改动,经过验证后再推到全局 `~/.claude/`。

## 为什么

直接编辑 `~/.claude/` 风险高:
- 一个坏权限可以让下次会话直接挂掉
- 一个坏 skill 会被静默加载,下次调用就中招
- 改坏了没法回滚 —— 你不一定记得自己改了什么

这个项目反过来走:任何改动先落到 `claude-lab/.claude/`,用户确认后写回全局。出问题时,`claude-archive/` 里有历次按日期归档的快照可以回滚。

## 结构

```
claude-lab/
├── .claude/
│   ├── CLAUDE.md           # 项目工作流契约 —— 从这里开始
│   ├── CLAUDE.zh.md        # 中文镜像
│   ├── settings.local.json # 项目独有权限(已 gitignore)
│   └── skills/             # in-flight 修改期间临时 checkout 的 skill
├── claude-archive/         # 按日期回滚快照(内容已 gitignore)
└── .gitignore
```

## 从哪开始

读 `.claude/CLAUDE.md`。这是工作流契约 —— 描述如何用本项目作为暂存区安全修改全局 Claude Code 配置,包括双语维护、in-flight tracker、归档规则。

## 双语约定

项目里 agent 可见的 markdown 文件保持英文源 + `.zh.md` 中文镜像同步。英文为权威版本。
