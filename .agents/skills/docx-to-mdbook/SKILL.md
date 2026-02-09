---
name: docx-to-mdbook
description: 将用户提供的 Word 文档转换处理为 Markdown 后提交到海塞姆 mdBook 文档仓库
metadata:
  audience: maintainers
  workflow: github
---

## 适用场景

- 收到 GitHub issue 要求添加 Word 文档到文档仓库
- 需要将 `.docx` 格式转换为 markdown
- 需要将转换后的文档整合到现有 mdBook 结构中
- 需要处理文档中的图像资源

## 前置条件

1. 遵循 `/io-workflow` 智能体技能
2. 已安装 `pandoc` 用于文档转换
3. 已安装 `git` 和 `gh` 用于版本控制和 GitHub 操作
4. 拥有对目标仓库的写入权限

### 环境准备

- 创建独立任务文件夹：`./<任务名>`
- 克隆目标仓库到任务文件夹内
- 下载 Word 文档附件（使用 curl 或 wget）

### 文档转换

- 使用 pandoc 将 `.docx` 转换为 mdBook 使用的 commonmark markdown：
  ```bash
  pandoc -f docx -t commonmark --extract-media=. -o "输出文件.md" "输入文件.docx"
  ```
- 提取的图像会保存在 `media/` 目录中

### 仓库整合

- 分析仓库结构（特别是 `src/` 目录和 `SUMMARY.md`）
- 创建合适的子目录（如 `src/products/产品名称/`）
- 创建资源目录（如 `src/assets/products/产品名称/`）
- 复制图像文件到资源目录
- 更新 markdown 中的图像引用路径：
  ```bash
  sed -i 's|\./media/|../../assets/products/产品名称/|g' 文档.md
  ```
- 递归更新上一级索引文件（如 `src/products/index.md`）
- 更新总目录文件（`src/SUMMARY.md`）

### 提交推送

- 添加相关更改到 git 暂存区；
- 按照提交规范创建提交：
  ```bash
  git commit -m "【文档】文档描述……"
  ```

## 注意事项

- Word 文档转换后可能需要清理格式（如删除无效的 `(\l)` 链接）
- 图像文件名可能为通用名称（image1.png），需注意命名冲突
- mdBook 项目需要保持 `SUMMARY.md` 的层级结构
- 提交消息应遵循项目规范（如 PJ568 提交规范）
