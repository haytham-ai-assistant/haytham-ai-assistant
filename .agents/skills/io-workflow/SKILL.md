---
name: io-workflow
description: 需获取用户上传文件、把文件（产物）发给用户或处理 GitHub 工单请求和项目任务时直接使用
---

- 任务流程
  - 任务前：
    1. 根据本流程创建详尽代办，包含任务前始中后的每一小步；
    2. 工单和仓库初始化：
       若需获取文件：
       - 引导用户前往[仓库工单](https://github.com/haytham-ai-assistant/haytham-ai-assistant/issues)发布任务信息并上传相关文件，询问用户是否已发；
       - 通过该工单获取相关文件；

       若需将产物呈现给用户，引导用户创建带 README 文件的新仓库供 Fork；

    3. 若存在名为“memory”的智能体技能（Agent Skills），让子智能体（Sub-agnet）命令检索或阅读后反馈有无历史解决方案；
    4. 向用户讨论确定具体执行方案并在代办上增加步骤。

  - 任务开始：
    - 创建新任务文件夹“`./<任务名>`”
    - 让子智能体初始化（Fork 或新建）远端 GitHub 任务仓库和任务文件夹内的本地 Git 仓库文件夹。
  - 任务中：
    - 时刻让子智能体同步发送进度评论信息至相关工单；
    - 每个独立更改都让子智能体根据 [PJ568 提交说明规范](https://github.com/PJ-568/git-commit-regulation)提交并推送。
  - 任务结束
    - 成功：
      1. 提交合并请求（Pull request）；
      2. 维护相关工单，关闭等。
      3. 虚心总结本次任务中摸索出的新方法和更准确或高效的差异，创建或更新智能体技能（和可选脚本）：
         仅记录与该任务
      4. 清理本次创建的任务文件夹和多余文件；
      5. 让子智能体在[“memory”智能体技能里](.agents/skills/memory/SKILL.md)更新本次任务简要相关内容：
         - 任务概要（一句话）
         - 日期时间
         - 相关技能（若有）
      6. 让子智能体合并缩减重复记忆；
    - 未成功：
      - 在工单内写清状态。

如果使用 HTTPS 远程仓库遇到认证问题，可使用 GitHub token：

```bash
git remote set-url origin https://${GH_TOKEN}@github.com/用户名/仓库.git
git push origin master
```
