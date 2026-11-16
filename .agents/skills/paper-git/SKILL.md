---
name: paper-git
description: 将当前项目与 GitHub 同步。指定 pull 拉取，或指定 push 提交并推送。
---

# Paper Git

调用 `$paper-git ,pull` 或 `$paper-git ,push 新增文档`。操作不明确时请用户指定。`push` 后的文字是提交说明，省略时根据差异拟写。提交说明格式为 `[YYYY-MM-DD]说明`，日期用上海时间。

先用 `git status -sb` 确认当前分支、同步目标和本地改动。实际调用即授权对应操作，示例只作说明。

## Pull

工作区干净时执行 `git pull --ff-only`。有未提交改动或分支已分叉时，停止并说明原因。

## Push

有新改动时，只暂存本次相关文件，核对待提交差异，再 `git commit`。

将当前分支推送到它配置的远端目标。已有本地提交且没有新改动时，直接推送。

简短报告同步结果。失败时说明原因。