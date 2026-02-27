---
name: github-io-workflow
description: 处理 GitHub 工单请求、文件获取与产物交付的标准工作流程
license: MIT
metadata:
  audience: maintainers
  workflow: github
  repository: haytham-ai-assistant
---

## 适用场景

- 需要从 GitHub 工单获取用户上传的文件
- 需要将任务产物（文件、代码等）通过 GitHub 交付给用户
- 处理 GitHub issue 中描述的任务请求
- 涉及 Fork、分支管理、Pull Request 的标准化操作

## 前置条件

1. **环境变量**：已配置 `GH_TOKEN` 或 `GITHUB_TOKEN`
2. **工具依赖**：已安装 `git` 和 `gh` CLI
3. **权限要求**：对目标仓库具有写入权限
4. **工作区**：在 `/workspace/<工作区名>/` 内执行操作

## 任务流程

### 任务前准备

#### 1. 创建详细待办清单

使用 `todowrite` 工具创建包含以下阶段的待办事项：

- 任务前准备（工单处理、环境初始化）
- 任务执行（开发、提交、推送）
- 任务收尾（PR 提交、清理、记忆更新）

#### 2. 工单和仓库初始化

**若需获取用户上传的文件：**

1. 引导用户前往 [仓库工单](https://github.com/haytham-ai-assistant/haytham-ai-assistant/issues) 创建 issue
2. 要求用户在 issue 中上传相关文件附件
3. 通过 issue 评论或附件下载链接获取文件
4. 询问用户是否已发布工单并确认文件可访问

**若需将产物交付给用户：**

1. 引导用户创建新仓库（包含基础 README 文件）
2. 获取新仓库的访问权限（用户邀请或公开仓库）
3. 或直接使用 `haytham-ai-assistant` 仓库的 issue 附件功能

#### 3. 历史方案检索

若存在名为 **`memory`** 的智能体技能（Agent Skills）：

1. 让子智能体读取 `.agents/skills/memory/SKILL.md`
2. 检索是否存在类似任务的历史解决方案
3. 反馈历史经验和可复用的方法

#### 4. 确定执行方案

与用户讨论并确认以下事项：

- 任务具体目标和交付物
- 技术方案和实现路径
- 时间预期和里程碑
- 特殊要求和注意事项

在待办清单上增加细化步骤。

### 任务开始

#### 1. 创建任务文件夹

```bash
mkdir -p /workspace/<工作区名>/<任务名>
cd /workspace/<工作区名>/<任务名>
```

#### 2. 初始化远端仓库

**使用 Fork 方式（推荐）：**

```bash
# Fork 上游仓库
retry-exec gh repo fork haytham-ai-assistant/haytham-ai-assistant --clone=false

# 创建功能分支
git checkout -b feature/<任务名>-<日期>
```

**新建仓库方式：**

```bash
# 创建新仓库
retry-exec gh repo create <仓库名> --public --source=. --remote=origin
```

#### 3. 初始化本地 Git 仓库

```bash
# 克隆到任务文件夹
git clone https://${GH_TOKEN}@github.com/haytham-ai-assistant/<仓库名>.git .

# 或切换到对应分支
git checkout -b feature/<任务名>-<日期>
```

### 任务执行

#### 1. 进度同步

- 每个关键节点都在相关 issue 中发表评论更新进度
- 使用 `gh issue comment <issue 号> --body "<进度内容>"`
- 遇到阻塞问题时及时在 issue 中说明

#### 2. 代码提交和推送

**每个独立更改都单独提交：**

```bash
# 添加更改
git add <文件>

# 按 PJ568 提交规范创建提交
git commit -m "【类型，范围】摘要"

# 推送更改
retry-exec git push origin feature/<任务名>-<日期>
```

**提交规范要点：**

- 类型：`初始`/`更改`/`新增`/`修复`/`格式`/`文档`/`测试`/`维护`
- 范围：模块名（可选）
- 摘要：≤30 汉字，使用祈使句
- 盘古之白：汉字与数字/字母/符号间加空格

详见 [PJ568 提交规范](https://github.com/PJ-568/git-commit-regulation)

### 任务收尾

#### 成功完成

1. **提交合并请求（Pull Request）**
   
   ```bash
   gh pr create --base main --head feature/<任务名>-<日期> \
     --title "<PR 标题>" \
     --body "<PR 描述>"
   ```

2. **维护相关工单**
   
   - 在 issue 中评论 PR 链接
   - 标记 issue 状态（如添加标签）
   - 任务完成后关闭 issue

3. **总结经验**
   
   若摸索出新方法或更高效的差异：
   
   - 创建或更新相关 Agent Skills
   - 记录任务特定经验

4. **清理任务文件夹**
   
   ```bash
   cd /workspace/<工作区名>/
   rm -rf <任务名>
   ```

5. **更新记忆技能**
   
   让子智能体在 `.agents/skills/memory/SKILL.md` 中添加记录：
   
   - 任务概要（一句话）
   - 日期时间
   - 任务仓库链接（若有）
   - 相关技能（若有）

6. **合并缩减重复记忆**
   
   定期整理记忆内容，避免冗余。

#### 未成功完成

- 在 issue 内写清当前状态和阻塞原因
- 记录已尝试的解决方案
- 提出需要的帮助或资源

## GitHub 认证

### 使用 HTTPS 远程仓库

若遇到认证问题，使用 GitHub Token：

```bash
# 设置远程仓库 URL（带 Token）
git remote set-url origin https://${GH_TOKEN}@github.com/用户名/仓库.git

# 推送
git push origin master
```

## 备用方案：GitHub API

当 `gh` CLI 工具因网络问题无法使用时，可直接使用 GitHub REST API：

### 创建 Pull Request

```bash
curl -X POST -H "Authorization: token ${GH_TOKEN}" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/组织名/仓库名/pulls \
  -d '{
    "title": "PR标题",
    "body": "PR描述",
    "head": "用户名:分支名",
    "base": "master"
  }'
```

### 检查状态

```bash
# 获取 check runs 状态
curl -H "Authorization: token ${GH_TOKEN}" \
  https://api.github.com/repos/组织名/仓库名/commits/提交SHA/check-runs

# 获取 PR 详情
curl -H "Authorization: token ${GH_TOKEN}" \
  https://api.github.com/repos/组织名/仓库名/pulls/PR编号
```

### 关闭 PR

```bash
curl -X PATCH -H "Authorization: token ${GH_TOKEN}" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/用户名/仓库名/pulls/PR编号 \
  -d '{"state":"closed"}'
```

### Token 安全注意事项

- **不要**在提交历史中暴露 Token
- **不要**将 Token 硬编码到代码中
- 使用环境变量或密钥管理工具
- 定期轮换 Token

## 使用示例

### 场景 1：处理用户上传的文档转换任务

**用户操作：**

1. 在 issue #123 上传 Word 文档
2. 描述要求转换为 Markdown

**智能体流程：**

1. 读取 issue #123 获取附件
2. 下载文档到任务文件夹
3. 使用 pandoc 转换
4. 提交转换结果
5. 在 issue 中评论 PR 链接

### 场景 2：交付代码产物

**用户操作：**

1. 创建 issue 描述需求
2. 等待智能体完成

**智能体流程：**

1. Fork 目标仓库
2. 创建功能分支
3. 实现功能并提交
4. 创建 Pull Request
5. 在 issue 中通知用户审查

## 注意事项

- **分支命名**：使用 `feature/<任务名>-<日期>` 格式
- **提交频率**：小步快跑，每个逻辑单元单独提交
- **PR 描述**：详细说明更改内容和测试方法
- **Issue 关联**：在 PR 中使用 `Closes #123` 自动关联关闭 issue
- **工作区清理**：任务完成后必须清理临时文件

## 相关资源

- [GitHub CLI 文档](https://cli.github.com/manual/)
- [GitHub Flow 工作流](https://docs.github.com/en/get-started/quickstart/github-flow)
- [PJ568 提交规范](https://github.com/PJ-568/git-commit-regulation)
