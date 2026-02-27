---
name: docx-to-mdbook
description: 将 Word 文档转换为 Markdown 并整合到海塞姆 mdBook 文档仓库
license: MIT
metadata:
  audience: maintainers
  workflow: github
  compatibility: github-io-workflow
---

## 何时使用

- 收到 GitHub issue 要求将 Word 文档添加到文档仓库
- 需要将 `.docx` 格式转换为 mdBook 兼容的 Markdown
- 需要将文档整合到现有 mdBook 结构中

## 工作流程

### 1. 环境准备

遵循 `/github-io-workflow` 智能体技能：

```bash
# 创建独立任务文件夹
mkdir -p ./<任务名>
cd ./<任务名>

# Fork 目标仓库（若未 Fork）
retry-exec gh repo fork haytham-ai/public-docs --clone

# 创建功能分支
cd public-docs
retry-exec git checkout -b feature/docx-<文档名称>-<日期>
```

根据对应仓库的自述，维护说明等确定之后的步骤格式。

### 2. 获取文档

```bash
# 从 issue 附件或指定位置下载 Word 文档
retry-exec curl -C -L <文档 URL> -o "原始文档.docx"
# 或使用 wget
retry-exec wget -c <文档 URL> -O "原始文档.docx"
```

### 3. 文档转换

使用 pandoc 转换为 mdBook 使用的 CommonMark 格式：

```bash
pandoc -f docx -t commonmark --extract-media=./media -o "文档.md" "原始文档.docx"
```

**参数说明**：
- `--extract-media=./media`：提取图像到 `media/` 子目录
- 输出格式为 commonmark，兼容 mdBook

### 4. 资源整合

假如是个关于产品的文档。

```bash
# 分析仓库结构
ls -la src/
cat src/SUMMARY.md

# 创建文档目录（按产品/模块分类）
mkdir -p src/products/产品名称/
mkdir -p src/assets/products/产品名称/

# 移动图像资源
cp -r media/* src/assets/products/产品名称/

# 更新图像引用路径
sed -i 's|!\[\](media/|![..](../../assets/products/产品名称/|g' "文档.md"
sed -i 's|!\[\](./media/|![..](../../assets/products/产品名称/|g' "文档.md"
```

### 5. 更新目录结构

**更新索引**（如 `src/products/index.md`）：
```markdown
- [产品名称](products/产品名称/index.md)
  - [新文档标题](products/产品名称/文档.md)
```

**更新总目录**（`src/SUMMARY.md`）：
```markdown
# 在对应章节添加
    - [文档标题](products/产品名称/文档.md)
```

### 6. 质量检查

```bash
# 检查 Markdown 语法
# 预览链接是否正确
grep -n "^\!\[" 文档.md

# 清理无效格式（Word 转换常见问题）
sed -i 's/{#.*}//g' 文档.md  # 删除 pandoc 生成的锚点
sed -i 's/(\l.*\.docx)//g' 文档.md  # 删除无效内部链接
```

等各类检查。

### 7. 提交推送

```bash
# 添加更改
git add src/products/产品名称/
git add src/assets/products/产品名称/
git add src/SUMMARY.md

# 创建提交（遵循 PJ568 提交规范）
git commit -m "【文档，产品】添加<文档名称>说明文档"

# 推送分支
retry-exec git push origin feature/docx-<文档名称>-<日期>
```

### 8. 创建 Pull Request

```bash
# 使用 gh CLI 创建 PR
retry-exec gh pr create \
  --base main \
  --title "【文档】添加<文档名称>" \
  --body "## 更改内容

- 转换并添加<文档名称>.md
- 添加相关图像资源
- 更新 SUMMARY.md 目录结构

## 来源

- Word 文档来源：<issue 链接或来源说明>
- 转换工具：pandoc

## 检查清单

- [x] Markdown 格式正确
- [x] 图像引用路径正确
- [x] 目录结构更新
- [x] 遵循提交规范"
```

## 注意事项

- Word 文档转换后可能需要清理格式（如删除无效的 `(\l)` 链接）
- 图像文件名可能为通用名称（image1.png），需注意命名冲突
- mdBook 项目需要保持 `SUMMARY.md` 的层级结构
- 提交消息应遵循项目规范（如 PJ568 提交规范）
- **文件验证**: 在转换前验证 Word 文档是否完整，使用 `unzip -t` 或 `python -m zipfile -t` 检查文件完整性
- **损坏文件处理**: 如果文档损坏无法转换，创建占位 Markdown 文件并保留原始文件，在工单中说明情况
- **网络问题**: 推送失败时尝试使用 SSH URL 或带 Token 的 HTTPS URL，并在工单中更新状态

## 实战经验补充

### GitHub 访问问题处理
当直接访问 `github.com` 超时或失败时，可使用以下镜像站进行 **读取操作**（克隆、下载文件、查看仓库）：

- **主用镜像**: `bgithub.xyz`（替换 `github.com`）
  ```bash
  # 下载 issue 附件（原始 URL: https://github.com/.../releases/download/...）
  retry-exec wget -c "https://bgithub.xyz/<owner>/<repo>/releases/download/..." -O "文档.docx"
  
  # 克隆仓库（带重试）
  retry-exec git clone "https://bgithub.xyz/haytham-ai/public-docs.git"
  ```

- **备用镜像**: `gitclone.com`（需调整 URL 格式）
- **注意**: 镜像站仅用于读取，**推送仍需使用原始 `github.com`**（可通过带 Token 的 HTTPS URL 绕过限制）

### 推送配置
当镜像站克隆后需切换回原始远程进行推送：

```bash
# 查看当前 remote
git remote -v

# 若为镜像站 URL，则修改 origin 为带 Token 的 HTTPS URL
git remote set-url origin "https://${GH_TOKEN}@github.com/haytham-ai-assistant/public-docs.git"

# 推送分支
retry-exec git push origin feature/...
```

### 完整性验证
转换前必须验证 Word 文档（`.docx` 为 ZIP 格式）是否完整：

```bash
# 方法一：使用 Python
python -m zipfile -t "文档.docx" && echo "文件完整" || echo "文件损坏"

# 方法二：使用 unzip
unzip -t "文档.docx"
```

若文件损坏，极可能是未完全下载，试着重新下载。
若确实为文件损坏，在工单中说明并请求重新上传。

### 重试机制
所有网络操作（下载、克隆、推送）必须使用 `retry-exec` 包装，确保单次失败不中断流程：

```bash
# 定义 retry-exec 函数（若未全局安装）
retry-exec() {
  local max_attempts=64
  local delay=1
  local attempt=1
  
  while [ $attempt -le $max_attempts ]; do
    if "$@"; then
      return 0
    fi
    
    echo "尝试 $attempt 失败，${delay}秒后重试..."
    sleep $delay
    attempt=$((attempt + 1))
  done
  
  echo "操作失败：$@"
  return 1
}
```

### 格式清理

Word 转换后常见问题：

1. **无效锚点**：删除 `{#section-name}` 格式
2. **内部链接**：Word 内部链接转换为 `.docx` 引用，需删除或修正
3. **样式残留**：删除 `::: {#custom-class}` 等格式块

### 图像处理

- 检查图像文件名是否冲突（`image1.png` 等通用名）
- 必要时重命名为有意义的名称
- 确保所有图像在 Markdown 中有正确的 `alt` 文本

### mdBook 结构

- 保持 `SUMMARY.md` 的缩进层级正确
- 新增文档必须在 `SUMMARY.md` 中引用，否则不会被构建

### 提交规范

遵循 PJ568 提交规范：

- 标题格式：`【类型，范围】摘要`
- 汉字与数字/字母/符号间加空格
- 单提交单目标

示例：
```
【文档，产品】添加 H800 产品规格说明
```
