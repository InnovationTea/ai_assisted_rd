# CLAUDE.md

## Project Overview

蓝区 AI 辅助研发工具集，存放可复用的 coding agent 技能和 bundled packages。技能可分发到其他项目供 Claude Code、Codex、OpenCode 使用。以 agent 驱动开发为主。

## Tech Stack

- Markdown / YAML（技能定义和元数据）
- JavaScript / Node.js 20.9+（bundled package 脚本）
- Git

## Critical Commands

```bash
rg --files                    # 列出所有文件
rg <pattern>                  # 搜索内容
node skills/agent-onboard/packages/git-code-tracker/install-to-project.js <project>  # 安装 tracker
node .claude/skills/ai-code-tracker/scripts/install.js --check  # 验证 tracker 安装
```

## Code Style & Conventions

- 提交信息：中文 Conventional Commit（`feat: 描述`、`fix: 描述`）
- 技能目录结构：`skills/<name>/SKILL.md` + `skills/<name>/agents/openai.yaml`
- 技能名称：kebab-case
- 文件内容语言：中文为主，技术术语保持英文

## Workflow Preferences

- 个人开发：直接 push main
- 团队协作：fork 后提 PR
- 变更前：读取目标文件理解上下文
- 变更后：自审无占位符/猜测内容，向开发者报告变更和验证结果
- 涉及项目结构或配置的修改需与开发者确认

## Architecture Notes

- `skills/` 是技能根目录，每个子目录是一个独立技能
- `skills/agent-onboard/packages/` 存放可分发的 bundled packages，版本由 `bundled-packages.json` 管控
- `skills/agent-onboard/references/` 是按需加载的参考文档，不要在生成文件中复制其内容
- `docs/superpowers/` 是历史文档，不再主动维护
- `agents.d/` 存放详细的知识蒸馏文件，`AGENTS.md` 保持简洁并指向它们
