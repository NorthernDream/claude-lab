# claude-lab —— 实验沙盒(中文翻译,Claude 不加载)

本项目用于试验 Claude Code 的配置:skills、hooks、permissions、agents、settings、workflow。目录名 `experiments/` 不是装饰。

## 改动契约

**核心规则:绝不直接编辑 `~/.claude/`。** 先在 `claude-lab/` 内验证、用户实测确认,再写回全局。

**双语维护:**对项目内任何 agent 可见的 markdown 文件(`CLAUDE.md`、`skills/<name>/SKILL.md` 等)的编辑,都要在同次 flow 内更新它的 `.zh.md` 同名文件 —— 英文为权威版本,`.zh.md` 镜像之。新文件:同次 flow 内从零创建对应 `.zh.md`。JSON / settings 文件不需要翻译。此条对下方所有 flow 都生效;flow 收尾时 `.zh.md` 没同步,这条 flow 就不算完成。

### 通过项目暂存修改全局内容

模式:在项目里 stage → 用户验证 → 写回全局 → 清理。setup 和 writeback 的具体动作取决于改的是什么内容。

1. **Set up 项目副本。**
   - **新增内容**(文件、章节、settings 字段):直接在项目里写 —— `.claude/skills/<name>/`、`.claude/CLAUDE.md` 横幅下方、`.claude/settings.local.json` 等。
   - **`~/.claude/skills/<name>/` 中已存在的 skill:**`cp -r ~/.claude/skills/<name> .claude/skills/<name>`。对插件 skill(位于 `~/.claude/plugins/cache/...`),通过 `find ~/.claude/plugins/cache -name '<name>' -type d` 定位实际路径并替换源路径。
   - **`~/.claude/CLAUDE.md` 中已存在的章节:**把章节文本复制到项目文件的横幅区(STAGED FOR GLOBAL 下方)。

2. **加一行 in-flight tracker** —— status=testing, scope=global-candidate, File=<项目位置>。skill checkout 把 Read-instead 设为 `.claude/skills/<name>/SKILL.md`;其他情况留空。

3. **改项目副本。**

4. **停下,让用户验证。**

   skill checkout 时:用 Read 工具读项目路径。**tracker 行存在期间不要调用 `Skill(<name>)`** —— 全局副本会静默赢。(2026-05-02 哨兵实测验证。`~/.claude/agents/<name>/` 下的 agent 很可能行为相同,首次用此路径处理 agent 前先跑哨兵测试。)

5. **用户确认** → tracker 那行 status 改为 `verified`。

6. **写回全局** —— 单次编辑,三个子动作:

   - 6a. **应用到全局:**
     - **新增内容:**`cp` 到 `~/.claude/`。
     - **已存在的 skill:**`rsync -a --delete .claude/skills/<name>/ ~/.claude/skills/<name>/`。`--delete` 关键 —— 项目副本里删掉的文件必须也从全局消失。(`cp *` 通配会留孤儿。)
     - **已存在的 CLAUDE.md 章节:**用项目版本替换 `~/.claude/CLAUDE.md` 中对应章节。
   - 6b. **从项目移除:**
     - **新增内容 / CLAUDE.md 章节:**把文件或章节文本从项目中删掉。
     - **已存在的 skill:**`rm -rf .claude/skills/<name>`。
   - 6c. **删除 tracker 那行。**

7. 归档到 `claude-archive/`。

### 修改 `~/.claude/settings.json` 中的全局权限

权限是叠加 —— `~/.claude/settings.json` 永远生效,`settings.local.json` 只能加,不能 shadow。项目暂存只覆盖"添加"。

1. 加一行 in-flight tracker(status=testing, scope=global-candidate, File=`.claude/settings.local.json`;Read-instead 留空)。
2. 把新增/改动后的权限加到 `claude-lab/.claude/settings.local.json`。
3. claude-lab 内实测。
4. **用户确认** → tracker 那行 status 改为 `verified`。
5. 先把权限加进 `~/.claude/settings.json` —— 顺序很重要:必须在撤本地之前完成,不然会出现"两边都没有该权限"的瞬间空档,撞上的命令会被拦。
6. 撤掉 `claude-lab/.claude/settings.local.json` 里冗余的本地条目,**删除 tracker 那行**。
7. 归档到 `claude-archive/`。

**撤销既有的全局权限**:没有项目级预演路径(因叠加性)。在用户**明确事前同意**下,直接编辑 `~/.claude/settings.json` 删除条目,然后归档。这种单步直接编辑不需要 tracker 行。

### 特殊情况
- **Plugin 提供的 skill**(如 `superpowers:*`,任何 `~/.claude/plugins/cache/...` 下的)必须走上面的 skill checkout 路径 —— 直接改会被插件升级覆盖掉。
- **没有项目级对应形式**(如 `~/.ssh/config` 别名、`~/.claude/settings.json` 里没有项目版本的字段如 `effortLevel`):明说,拿到用户确认,再动全局。tracker 里以 `out-of-scope` 标记并写原因。
- **逃生口 —— 文档微改**:`~/.claude/` 下纯 typo / 描述微调,用户**明确同意**"跳过流程",可直接编全局。默认还是走 checkout。
- **放弃路径**:流程中任意阶段,回滚项目改动、删 tracker 行、不归档。skill checkout 还要 `rm -rf .claude/skills/<name>` 清掉副本。

### 归档
任意流程走完(包括最后删 tracker 行)后,把当前 `.claude/` 快照到 `claude-archive/`:
```bash
cp -r .claude claude-archive/$(date +%Y-%m-%d)-<简短标签>
```
标签描述本次快照的变化(如 `promotion-contract`、`redis-cluster-v2`)。这是回滚保险。**仅当**`.claude/` 与最新一份快照字节级一致时才跳过。

## In-flight tracker

单表追踪所有 in-flight 改动到 `.claude/`。空 = 当前没有实验在跑。行进入 `promoted`/`accepted`/`abandoned` 即删。

**`Read-instead`** 列只对 skill checkout 行填 —— 它指示 agent 用 `Read` 读项目副本,**不要**调 `Skill(<name>)`(全局会静默赢)。其他行此列留空。

**不归本表管:**`.claude/` 之外的改动(如 `~/.ssh/config` 别名、`~/.ssh/id_*` 密钥),被改到时在对话里点名,不进表格。

**Scope:** `local-only`(项目绑死)| `global-candidate`(将来推全局)
**Status:** `testing`(刚落地)→ `verified`(用户确认)→(`promoted`(global-candidate)/ `accepted`(local-only)/ `abandoned`(任意)⇒ 删行)

| Entry | File | Scope | Read-instead | Status | Notes |
|---|---|---|---|---|---|
| *(空 —— 当前没有 in-flight 工作)* | | | | | |

<!-- STAGED FOR GLOBAL: 此横幅下方直至文件末尾的所有内容,经用户同意后推到 ~/.claude/CLAUDE.md。
     横幅本身**不**复制,只复制下方的标题和正文。文件约定:项目独有内容(workflow contract、
     in-flight tracker 等)放在横幅**上方**;待推全局的内容放在横幅**下方**。已有章节的提级
     (项目→全局候选)或降级(反向)= 把它跨越本横幅移动即可,无需其他簿记。 -->
