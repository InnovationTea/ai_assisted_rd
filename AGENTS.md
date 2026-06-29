# AGENTS.md

## Project Snapshot

蓝区 AI 辅助研发工具集。存放可复用的 coding agent 技能（skills）、bundled packages 和流程规范文档。产出的技能可分发到其他项目，供 Claude Code、Codex、OpenCode 等 agent 平台使用。

## Tech Stack

- **语言**: Markdown（技能定义）、YAML（Codex UI 元数据）、JavaScript/Node.js（bundled package 脚本）
- **Agent 平台**: Claude Code、Codex (OpenAI)、OpenCode
- **版本管理**: Git
- **运行时依赖**: Node.js 20.9+（bundled package 安装脚本需要）

## Commands

无传统 install/build/test 命令。项目产出是技能文档和脚本，不是可编译的应用。

| 操作 | 命令 |
|------|------|
| 列出所有文件 | `rg --files` |
| 搜索内容 | `rg <pattern>` |
| 安装 git-code-tracker 到目标项目 | `node skills/agent-onboard/packages/git-code-tracker/install-to-project.js <target-project>` |
| 验证 git-code-tracker 安装 (Claude) | `node .claude/skills/ai-code-tracker/scripts/install.js --check` |

## Environment Setup

- Node.js 20.9+（用于 bundled package 脚本）
- Git
- 无需额外环境变量或本地服务

## Approved Skills And Tools

见 [agents.d/tooling.md](agents.d/tooling.md)。

## agents.d Index

| 文件 | 内容 |
|------|------|
| [tooling.md](agents.d/tooling.md) | 已有技能清单、bundled packages、平台技能 |
| [development-loop.md](agents.d/development-loop.md) | 技能开发流程和验证 |
| [change-recipes.md](agents.d/change-recipes.md) | 常见变更操作步骤 |
| [review-handoff.md](agents.d/review-handoff.md) | 人工审查移交标准 |

## Repository Map

```
.
├── AGENTS.md                  # Agent 入口指南（本文件）
├── CLAUDE.md                  # Claude Code 平台配置
├── agents.d/                  # 详细知识蒸馏文件
├── skills/                    # 技能目录
│   ├── agent-onboard/         # Agent 上手技能（含 bundled packages）
│   │   ├── SKILL.md
│   │   ├── agents/openai.yaml
│   │   ├── references/        # 按需加载的参考文档
│   │   ├── packages/          # 可分发的 bundled packages
│   │   │   └── git-code-tracker/
│   │   └── bundled-packages.json
│   └── gitpush/               # Git 提交推送技能
│       ├── SKILL.md
│       └── agents/openai.yaml
├── docs/superpowers/          # 历史设计文档（不再主动维护）
│   ├── specs/
│   └── plans/
├── .claude/                   # Claude Code 项目配置
└── .agents/                   # 预留目录
```

## Development Rules

- **提交信息**: 中文 Conventional Commit 格式（`feat: 描述`、`fix: 描述`、`chore: 描述`）
- **技能结构**: 每个技能包含 `SKILL.md`（定义和工作流）+ `agents/openai.yaml`（Codex UI 元数据）
- **分支策略**: 个人开发直接 push main；团队协作必须 fork 后提 PR
- **变更确认**: 涉及项目的修改需要和开发者确认后再执行

## Testing and Verification

项目无自动化测试套件。验证方式：

1. 技能文件结构完整性：确认 `SKILL.md` 和 `agents/openai.yaml` 存在且格式正确
2. 内容自审：检查无占位符（TODO/TBD）、无猜测性命令、无泛泛建议
3. Bundled package 安装验证：使用 `--check` 参数验证安装状态

## Agent Workflow

1. 读取 `AGENTS.md` 和相关 `agents.d/` 文件了解项目上下文
2. 用 `rg --files` 和 `rg` 定位相关文件
3. 阅读目标文件后再编辑
4. 保持变更聚焦，不扩大修改范围
5. 遵循项目已有的风格和约定
6. 完成后向开发者报告变更内容和验证结果

## Human Review Handoff

见 [agents.d/review-handoff.md](agents.d/review-handoff.md)。

## Risk Areas

- `skills/agent-onboard/bundled-packages.json` — 控制 bundled package 版本和安装策略，修改需确认
- `skills/agent-onboard/packages/` — 已版本锁定的外部包，不要随意修改内部文件
- `skills/*/SKILL.md` — 技能核心定义，修改会直接影响 agent 行为
- `.claude/settings.json` / `.claude/settings.local.json` — Claude Code 配置，修改影响权限和 hooks

## Do Not

- 不要修改 `packages/` 下 bundled package 的源文件，除非明确要升级版本
- 不要猜测不存在的命令或工具
- 不要将推断写成事实
- 不要在 onboarding 文件中写入密钥、个人路径或临时信息
- 不要覆盖已有的 `AGENTS.md` 或 `CLAUDE.md` 而不先确认
- 不要在没有确认的情况下运行安装脚本或修改 git hooks

## Missing Context

- 团队协作 PR 审查的具体标准尚未确定
- 是否有 CI/CD 流水线待确认
- `docs/superpowers/` 的长期规划未明确（当前为历史设计文档，不再主动维护）

<!-- Codex: prefer rg over find; read nearby code before editing; keep changes scoped -->
## AI Code Tracker

Before modifying code in this repository, load the opencode skill `ai-code-tracker` and run its preflight check. If tracking is not installed or is broken, ask the user whether to install or repair it. If the user confirms, run the project-level install or repair script automatically, rerun preflight, and continue with code changes only after preflight passes.

After installing or repairing ai-code-tracker, tell the user to restart the current opencode session because project plugins are loaded at opencode startup.

When cherry-picking commits, always use `git cherry-pick -x` to preserve the source commit reference. This allows ai-code-tracker to copy the original AI line statistics into the cherry-picked commit's tracking record.
