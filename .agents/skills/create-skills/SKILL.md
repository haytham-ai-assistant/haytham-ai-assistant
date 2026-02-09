---
name: create-skills
description: 据最新智能体技能格式创建新技能，当创建新智能体技能（Agent Skills）时使用
---

## 文件放置

每个技能名称创建一个独立文件夹，并在其中放置 `SKILL.md` 文件：

- 工作区智能体兼容路径：`./.agents/skills/<英文名称>/SKILL.md`
- 可选脚本路径：`./.agents/skills/<英文名称>/scripts/some.sh`

## 编写前置元数据（Frontmatter）

每个 `SKILL.md` 文件必须以 YAML 前置元数据开头。
仅识别以下字段：

- `name`（必需）
- `description`（必需）
- `license`（可选）
- `metadata`（可选，字符串到字符串的映射）

未知的前置元数据字段将被忽略。

## 名称验证规则

`name` 字段必须满足：

- 长度为 1–64 个字符
- 仅包含小写字母、数字，使用单个连字符作为分隔符
- 不得以 `-` 开头或结尾
- 不得包含连续的 `--`
- 必须与包含 `SKILL.md` 的目录名称一致
- 等效正则表达式：
  ```
  ^[a-z0-9]+(-[a-z0-9]+)*$
  ```

## 描述长度规则

- `description` 长度必须为 1–1024 个字符
- 描述应足够具体，以便智能体能够正确选择该技能

## 示例

仅作示例。

`.agents/skills/git-release/SKILL.md`：

```plaintext
---
name: git-release
description: 创建版本发布和更新日志
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---

## 怎么做

- 根据已合并的和并请求起草发布说明
- 提出版本升级建议
- 提供可直接复制粘贴的 `gh release create` 命令

## 何时使用

在准备带标签的发布版本时使用。如果目标版本策略不明确，请提出澄清性问题。
```
