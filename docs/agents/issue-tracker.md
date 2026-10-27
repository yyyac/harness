# Issue tracker：GitHub

本仓库的问题（issue）与规格（spec）存放在 GitHub Issues 中。所有操作使用 `gh` CLI。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行正文用 heredoc。
- **读取 issue**：`gh issue view <number> --json number,title,body,labels,comments`。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，配合必要的 `--label` 与 `--state` 过滤。
- **将 issue 设为某个父 issue 的子 issue**：`gh issue create --parent <parent> ...`，或事后 `gh issue edit <parent> --add-sub-issue <child>`（需要 `gh` 2.94+）。旧版 `gh`：`gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（使用数据库 id，见下文 **阻塞关系**）。若子 issue 不可用，在子 issue 正文顶部写 `Part of #<parent>`。
- **评论**：`gh issue comment <number> --body "..."`。
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`。
- **关闭**：`gh issue close <number> --comment "..."`。

仓库由 `gh` 在克隆目录内自动从 `git remote -v` 推断。

## 以 PR 作为 triage 入口

**PR 作为请求入口：否。** _（若本仓库把外部 PR 当作功能请求处理，改为 `是`；`/triage` 会读取此开关。）_

设为 `是` 后，PR 与 issue 走相同的标签与状态流程，使用等价的 `gh pr` 命令：

- **读取 PR**：`gh pr view <number> --comments`，diff 用 `gh pr diff <number>`。
- **列出待 triage 的外部 PR**：`gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **评论 / 打标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 issue 与 PR 共用一个编号空间，裸写 `#42` 可能指两者之一：先用 `gh pr view 42` 解析，失败再退回 `gh issue view 42`。

## 当某个 skill 要求「发布到 issue tracker」

创建一个 GitHub issue。

## 当某个 skill 要求「取回对应的 ticket」

按上文 **读取 issue** 的方式读取。

## Wayfinding 操作

供 `/wayfinder` 使用。**map** 是一个 issue，**子票** 是它的子 issue。

- **Map**：一个打 `wayfinder:map` 标签的 issue，承载 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
- **子票**：通过 GitHub 原生 sub-issue 关联到 map（见 **将 issue 设为某个父 issue 的子 issue**）。若尚未启用 sub-issue，把子 issue 加入 map 正文的任务列表，并在子 issue 正文顶部写 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。认领后把票分配给执行者。
- **阻塞关系**：使用 GitHub 的**原生 issue dependencies**——这是规范且 UI 可见的表达。用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加边，其中 `<blocker-db-id>` 是阻塞者的数字**数据库 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，_不是_ `#number`，也不是 `node_id`）。GitHub 通过 `issue_dependencies_summary.blocked_by` 报告（只计未关闭的阻塞者，即实时门禁）。若依赖功能不可用，退回在子 issue 正文顶部写 `Blocked by: #<n>, #<n>`。所有阻塞者关闭后，该票即解阻。
- **Frontier 查询**：列出 map 的未关闭子 issue（`gh issue list --state open`，范围限定为该 map 的 sub-issue / 任务列表），剔除存在未关闭阻塞者的（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中列出未关闭 issue）以及已有 assignee 的；按 map 中顺序取第一个。
- **认领**：`gh issue edit <n> --add-assignee @me`，作为本次会话的第一次写操作。
- **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，再向 map 的 Decisions-so-far 追加一条上下文指针（要点 + 链接）。
