# Tooling

## 已有技能

### agent-onboard

- **路径**: `skills/agent-onboard/`
- **用途**: 扫描项目、访谈项目所有者、生成 AGENTS.md / agents.d / CLAUDE.md 等 agent 上手文件
- **触发**: 用户要求 onboard agent、生成 AGENTS.md、使项目 AI-ready
- **平台**: Claude Code（`/agent-onboard`）、Codex（`$agent-onboard`）
- **输入**: 目标项目路径（默认当前目录）
- **必读**: `SKILL.md`，按阶段按需读取 `references/` 下的参考文件
- **输出**: `AGENTS.md`、`agents.d/`、`CLAUDE.md` 等文件
- **安全级别**: 自主执行扫描和生成，安装 bundled package 需用户确认

### gitpush

- **路径**: `skills/gitpush/`
- **用途**: 提交本地变更并推送到远端
- **触发**: 用户要求 gitpush、提交并推送、`/gitpush`
- **平台**: Claude Code（`/gitpush`）、Codex（`$gitpush`）
- **输入**: 无（操作当前工作目录）；可选 `--squash` 参数
- **输出**: 提交信息、变更文件数、推送结果
- **安全级别**: 自主执行（已被用户显式调用）
- **注意**: 提交信息使用中文 Conventional Commit 格式；squash 模式会 force-with-lease 推送

## Bundled Packages

### git-code-tracker (v1.0.1)

- **包路径**: `skills/agent-onboard/packages/git-code-tracker/`
- **来源**: `https://github.com/yooocen/git-code-tracker.git` @ `v1.0.1` (`e1cc62d`)
- **用途**: 通过 git hooks 和 AI 工具钩子自动追踪每次 commit 中 AI 生成的代码行数
- **支持平台**: Claude Code、OpenCode
- **安装命令**: `node skills/agent-onboard/packages/git-code-tracker/install-to-project.js <target-project>`
- **安装写入**: `.opencode/skills/ai-code-tracker`、`.claude/skills/ai-code-tracker`、`.git/hooks/`、`.ai-tracking/`、`.gitignore`、`AGENTS.md`
- **验证**: `node .claude/skills/ai-code-tracker/scripts/install.js --check`（Claude）或 `node .opencode/skills/ai-code-tracker/scripts/install.js --check`（OpenCode）
- **安全级别**: 安装需用户确认（会修改项目文件和 git hooks）
- **前置条件**: Node.js 20.9+
- **不要**: 手动在 commit message 中加 `[ai-tracking]`（追踪器自动处理）
