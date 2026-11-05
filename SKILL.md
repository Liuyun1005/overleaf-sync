---
name: overleaf-git-sync
description: 安装和配置 Git，引导获取 Overleaf Git 认证令牌，将 Overleaf 项目克隆或同步到本地，以及提交并上传指定文件。适用于 Overleaf Git 连接、认证失败、拉取更新和推送论文源码；不自动编译或修改论文内容。
---

# Overleaf Git 同步

用中文引导用户完成所请求的阶段。先读目标目录的 AGENTS.md，确认项目、目录、远端和同步方向；已有授权继续有效。创建技能或索取教程本身不授权实际安装软件或上传论文。不要把某个项目允许丢弃本地修改的约定推广到其他项目。

## 1. 安装与设置 Git

先执行 `git --version`。Windows 若命令不存在，检查已安装程序和 PATH，避免重复安装。需要安装时使用 Git 官方安装包，或：

```powershell
winget install --id Git.Git -e --source winget
```

winget 包由社区维护；官方安装入口见文末。安装后重新打开终端并验证版本。macOS/Linux 使用相应平台官方安装说明，不在已有 Git 时升级整个系统。

提交前检查 `git config --get user.name` 和 `git config --get user.email`；缺失时让用户提供提交署名，不猜测身份。优先在仓库内设置，仅在用户希望所有仓库共用时使用 `--global`：

```powershell
git config --local user.name '用户提供的姓名'
git config --local user.email '用户提供的邮箱'
git config --show-origin --get-all credential.helper
```

Git 提交署名与 Overleaf 登录身份不同。不要无故修改全局换行符、代理或凭据配置。Windows 优先复用 Git Credential Manager/系统凭据存储；先检查 `git credential-manager --version`，只有已安装且需要设置时才按该版本官方说明配置。不要使用明文 `credential.helper store` 保存 token。

## 2. 引导获取远端地址和“密码”

Overleaf Cloud 的 Git 集成需要项目具备对应权益；学校订阅可能提供权益，以项目实际可见功能为准。Server Pro 还取决于管理员配置。Git 同步与 GitHub 同步不是同一功能，不要求用户创建 GitHub 仓库。

1. 请用户登录 Overleaf 并打开目标项目，在左侧 **Integrations → Git** 中复制 Git 地址；旧界面可能在 Menu 中。优先复用登录会话；操作浏览器时遵守可用浏览器工具的使用说明。
2. 在 Git 对话框选择 **Generate token**，或进入 [Account Settings](https://www.overleaf.com/user/settings) 的 Git authentication tokens 区域生成令牌。
3. 认证用户名为 `git`；Password 提示处输入该 token，**不是网页登录密码**。旧 token 全文不能再次显示，遗失时生成新的 token。学校 SSO 用户也使用 token。
4. 由用户直接在本机交互式终端或凭据窗口输入 token，不要求粘贴到聊天。若工具无法安全提供交互输入，给出需在本机执行的命令，待用户完成后继续验证。不要把 token 写进 URL、命令参数、脚本、日志或技能文件；不要读取或输出系统保存的凭据。

Cloud 地址通常为 `https://git@git.overleaf.com/项目ID`；自托管地址从实际界面复制，不能硬套 Cloud 域名。检查已有 remote 时避免输出可能嵌入的密码，报告中脱敏。页面入口、权益和 token 有效期可能变化，遇到不一致时查文末官方说明。

## 3. 首次获取文件

将以下示例中的地址和目录替换成已确认的值；不要直接执行示例占位内容。

```powershell
git clone 'https://git@git.overleaf.com/项目ID' '目标目录'
Set-Location '目标目录'
git status --short
git branch -vv
git remote -v
git ls-remote --symref origin HEAD
git rev-parse HEAD
```

目标应为不存在或空目录；已有文件时先检查，不覆盖，也不在已有仓库内重新 `git init`。已有本地论文但没有远端历史时，先将 Overleaf 克隆到独立目录，再把用户指定源码复制进去、审查差异并提交；不要上传外层项目或 `.git`。

检查实际分支：新项目通常为 `main`，旧克隆可能为 `master`；不能静默切换分支或替换已有 remote。保留嵌套仓库历史，在论文子仓库内运行 Git。下文命令假定已核实远端 `origin` 和分支 `main`，否则替换成实际值。

## 4. 获取远端更新

每条命令成功后才进行依赖它的下一步；PowerShell 不应在失败后继续执行写操作。

```powershell
git status --short
git branch -vv
git rev-parse HEAD
git fetch origin
git log --oneline HEAD..origin/main
git diff --stat HEAD origin/main
git diff --name-status HEAD origin/main
```

`fetch` 只获取远端引用，不更新本地文件。记录起始提交，检查本地修改、未跟踪文件和本地独有提交。工作树干净且 HEAD 为远端祖先时：

```powershell
git merge --ff-only origin/main
```

若仅本地领先，不必合并；“下载更新”不授权上传这些提交。若存在本地修改或分叉，默认保留，检查差异后按用户意图处理；不要自动 stash、reset、clean 或覆盖冲突。可以先用独立快照检查远端。只有用户或适用项目规则已明确授权以远端覆盖时，才在核实目录、remote、分支和将丢弃内容后使用 `git reset --hard origin/main`。不要清理无关未跟踪文件。审核任务有更严格的只读要求时遵守该要求。

验证 HEAD、远端 SHA、`git diff --exit-code origin/main --` 和 `git status --short`。完全镜像同步应提交相同且已跟踪文件无差异；保留本地内容的同步须明确说明差异。同步源码不授权编译、改写正文或推送。

## 5. 提交与上传

用户明确要求上传时执行，无需重复索取已有授权。先 fetch 并检查远端新变化，再编辑；若上传的是已有本地改动，保留这些待上传改动，不套用“下载时丢弃本地”的规则。合并并发变化时保留双方内容，技术含义不明确的冲突交给用户决定。

```powershell
git diff --check
git diff -- main.tex
git add -- main.tex
git diff --cached --stat
git diff --cached
git commit -m 'paper: update authorized manuscript content'
git push origin HEAD:main
git rev-parse HEAD
git ls-remote origin refs/heads/main
git status --short
```

将 `main.tex` 换为实际授权文件列表。先检查暂存区，若已有不相关内容，不将其混入本次提交，也不擅自撤销。只暂存具名文件，不默认 `git add .`；不上传凭据、缓存、编译产物、审稿 JSON 或外层 AGENTS.md。没有变化时不创建空提交。

禁止 force push。若远端推进导致拒绝，重新 fetch、检查并协调并发修改后再普通 push；无法安全解决时报告具体冲突，停止推送重试。最终核对本地提交与远端分支 SHA；若验证时远端又推进，检查本次提交是否已成为远端祖先，准确报告。

## 6. 常见障碍与交付

- Authentication failed：核实用户名 `git`、token 有效性及缓存的旧凭据；由用户在系统凭据管理界面更新对应 Overleaf 条目，不清空所有凭据。
- Repository not found / no git access：核对 Git 地址、项目权限与权益，不将所有错误都当成密码错误。
- 网络/TLS 失败：报告具体错误，检查网络或机构代理；不要关闭 TLS 验证。
- non-fast-forward：fetch 后检查分叉，不能通过强推解决。
- Overleaf 不支持 Git LFS、远端多分支及仓库内部的 submodule；上传失败时检查实际文件限制。存在活跃批注或 Track Changes 时说明 Git 上传可能影响它们，尤其文件移动/改名，按用户意图处理。

完成后简洁报告 Git 是否可用、项目目录、远端（无秘密）、分支、前后提交、变更文件和验证结果；明确保留或丢弃了哪些本地内容。认证未完成就说明待完成步骤，不把 fetch 成功说成文件已同步，也不把 commit 成功说成已上传。

## 官方参考

核对日期：2026-11-05。认证、权益、平台安装与限制变化时重新查询：

- [Git 安装（Windows）](https://git-scm.com/install/windows)
- [Git 各平台安装](https://git-scm.com/install/)
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager)
- [Overleaf Git token](https://docs.overleaf.com/integrations-and-add-ons/git-integration-and-github-synchronization/git-integration/git-integration-authentication-tokens)
- [Overleaf Git 集成与限制](https://docs.overleaf.com/integrations-and-add-ons/git-integration-and-github-synchronization/git-integration)
