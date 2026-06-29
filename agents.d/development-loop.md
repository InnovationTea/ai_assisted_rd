# Development Loop

## 技能开发流程

本项目以 agent 驱动开发为主：用户提需求，agent 实现和自验证。

### 新建技能

1. 在 `skills/` 下创建技能目录：`skills/<skill-name>/`
2. 创建 `SKILL.md`（包含 frontmatter 和工作流定义）
3. 创建 `agents/openai.yaml`（Codex UI 元数据）
4. 按需创建 `references/`、`scripts/`、`assets/`、`packages/` 子目录
5. 验证文件结构完整性

### SKILL.md 结构

```yaml
---
name: <skill-name>
description: <触发条件描述>
---
```

后跟 Markdown 格式的工作流指令。

### agents/openai.yaml 结构

```yaml
interface:
  display_name: "<显示名称>"
  short_description: "<简短描述>"
  default_prompt: "<默认触发提示>"
```

### 修改已有技能

1. 读取当前 `SKILL.md` 理解现有逻辑
2. 按需读取 `references/` 下的参考文件
3. 做聚焦修改，不扩大范围
4. 确认无占位符、无猜测性内容

## 验证清单

- [ ] `SKILL.md` frontmatter 完整（name、description）
- [ ] `agents/openai.yaml` 格式正确
- [ ] 无 TODO/TBD 占位符
- [ ] 无猜测性命令或泛泛建议
- [ ] 推断内容标注为推断，未写成事实
- [ ] 引用的文件路径实际存在

## 提交

使用中文 Conventional Commit 格式：

```
feat: 添加新技能 xxx
fix: 修复技能 xxx 的工作流问题
chore: 更新技能元数据
refactor: 优化技能结构
docs: 补充设计文档
```
