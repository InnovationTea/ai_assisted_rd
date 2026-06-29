# Change Recipes

## 添加新技能

**涉及文件**:
- 新建 `skills/<name>/SKILL.md`
- 新建 `skills/<name>/agents/openai.yaml`
- 可选：`references/`、`scripts/`、`assets/`、`packages/`

**步骤**:
1. 确定技能名称（kebab-case）
2. 编写 `SKILL.md`，包含 frontmatter（name、description）和工作流
3. 编写 `agents/openai.yaml`，包含 display_name、short_description、default_prompt
4. 验证文件结构
5. 提交：`feat: 添加 <name> 技能`

## 修改技能工作流

**涉及文件**:
- `skills/<name>/SKILL.md`

**步骤**:
1. 读取当前 SKILL.md
2. 理解现有工作流逻辑
3. 做聚焦修改
4. 自审：无占位符、无猜测命令
5. 提交：`fix:` 或 `refactor:` + 描述

## 添加 bundled package

**涉及文件**:
- 新建 `skills/<skill>/packages/<package-name>/`
- 更新 `skills/<skill>/bundled-packages.json`
- 可能更新 `skills/<skill>/SKILL.md`

**步骤**:
1. 将 package 放入 `packages/` 目录
2. 在 `bundled-packages.json` 中注册：name、version、source、path、platform_skills、safety
3. 在 SKILL.md 中更新 bundled package 文档
4. 提交：`feat: 添加 bundled package <name>`

## 升级 bundled package 版本

**涉及文件**:
- `skills/<skill>/packages/<package-name>/` 内容替换
- `skills/<skill>/bundled-packages.json` 版本和 commit 更新

**步骤**:
1. 与开发者确认升级目标版本
2. 替换 packages 目录内容
3. 更新 `bundled-packages.json` 中的 version、ref、commit
4. 验证安装脚本可用
5. 提交：`chore: 升级 <package-name> 到 <version>`

## 更新 onboarding 文件

**涉及文件**:
- `AGENTS.md` 和/或 `agents.d/*.md` 和/或 `CLAUDE.md`

**步骤**:
1. 读取现有文件
2. 确定新知识归属哪个文件
3. 做最小连贯更新，不重新生成整个文件
4. 确认无重复或矛盾内容
5. 提交：`docs: 更新 agent 上手文档`
