# GitKraken 6.5.1 使用手册

> 🦑 最强 [[../../javaEE/Git]] 图形化客户端 · 可视化分支管理 · 高效协作开发
>
> **平台支持**：Windows / macOS / Linux
> **版本特性**：免费版支持私有仓库（6.5.1 为最后一个免费支持私有仓库的版本）
> **平台集成**：GitHub / GitLab / Bitbucket / Azure DevOps

---

## 目录

1. [软件简介](#1-软件简介)
2. [安装与配置](#2-安装与配置)
3. [界面总览](#3-界面总览)
4. [仓库管理](#4-仓库管理)
5. [提交与暂存](#5-提交与暂存)
6. [分支管理](#6-分支管理)
7. [合并与变基](#7-合并与变基)
8. [远程操作](#8-远程操作)
9. [冲突解决](#9-冲突解决)
10. [Pull Request 管理](#10-pull-request-管理)
11. [Stash 储藏](#11-stash-储藏)
12. [SSH 密钥配置](#12-ssh-密钥配置)
13. [GitFlow 工作流](#13-gitflow-工作流)
14. [快捷键速查](#14-快捷键速查)
15. [常见问题（FAQ）](#15-常见问题faq)

---

## 1. 软件简介

GitKraken 是一款跨平台的 [[../../javaEE/Git]] 图形化客户端，由 Axosoft 开发。它以直观的可视化分支图、流畅的交互体验和强大的协作功能著称，是目前最受欢迎的 [[../../javaEE/Git]] GUI 工具之一。

> **💡 关于 6.5.1 版本**
>
> GitKraken 6.5.1 是**最后一个免费支持私有仓库**的版本。从 6.5.2 开始，免费版仅能打开公开仓库，私有仓库需要付费 Pro 许可证。如果你需要免费管理私有仓库，请保留使用此版本。

### 核心特性

| 特性 | 说明 |
|------|------|
| **可视化分支图** | 以 DAG 有向无环图动态渲染提交网络，分支以彩色线条标识，合并点高亮显示 |
| **拖拽式分支操作** | 支持拖拽合并、变基、检出分支，右键菜单提供回退、创建标签、Cherry-pick 等操作 |
| **精细化暂存控制** | 双栏分层结构，支持逐行/逐块暂存（Stage Hunk），彻底取代 `git add -p` |
| **内置合并工具** | 三向合并视图，支持逐行选择冲突解决方案，可视化对比 |
| **平台集成** | 原生支持 GitHub、GitLab、Bitbucket、Azure DevOps，可直接管理 PR/MR |
| **大型仓库优化** | 增量索引、异步加载、内存缓存淘汰机制，数万次提交仍保持流畅 |

### 系统要求

| 操作系统 | 最低版本 | 架构 | 备注 |
|----------|----------|------|------|
| Windows | Windows 10 / Server 2016 | 64 位 | 需要 .NET Framework 4.6+ |
| macOS | 10.13 (High Sierra) | Intel / Apple Silicon | 支持 M1 芯片（Rosetta 2） |
| Linux (Ubuntu) | 16.04 LTS | 64 位 | 提供 .deb / .rpm / .tar.gz |

> **ℹ️ 前置条件**
>
> GitKraken 底层调用系统 [[../../javaEE/Git]] 可执行文件，建议提前安装 Git CLI（2.11+）并配置好 `user.name` 和 `user.email`。

---

## 2. 安装与配置

### 2.1 下载安装

#### Windows

1. **下载安装包**：获取 `GitKrakenSetup-6.5.1.exe` 安装程序（约 113MB）
2. **运行安装**：双击运行安装程序，按照向导点击 Next 完成安装。默认安装路径为 `C:\Users\用户名\AppData\Local\GitKraken`
3. **启动 GitKraken**：首次启动会要求登录 GitKraken 账号（可用 GitHub / GitLab / Bitbucket / Google 账号快捷登录）

#### macOS

1. **下载 .dmg 文件**：获取 `GitKraken-v6.5.1-macOS.dmg`
2. **拖拽安装**：打开 .dmg，将 GitKraken 拖入 Applications 文件夹
3. **首次启动**：在启动台打开 GitKraken。如提示"来自未知开发者"，右键选择"打开"即可

#### Linux (Ubuntu/Debian)

```bash
# 下载 .deb 包
wget https://release.gitkraken.com/linux/gitkraken-amd64.deb

# 安装
sudo dpkg -i gitkraken-amd64.deb

# 如有依赖缺失，修复依赖
sudo apt-get install -f

# 启动
gitkraken
```

### 2.2 首次配置

#### 配置 Git 用户信息

首次使用前，确保 Git 全局用户信息已正确设置：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
```

也可以在 GitKraken 内部配置：`File → Preferences → Git`

#### 配置默认编辑器

在 `Preferences → General` 中可设置外部编辑器（如 VS Code、Sublime Text 等），用于打开和编辑文件。

#### 配置主题

GitKraken 支持亮色和暗色两种主题：`Preferences → UI Customization → Theme`

- **Light**：浅色主题，适合白天使用
- **Dark**：深色主题（默认），护眼且美观

### 2.3 关联 Git 托管平台

在 `Preferences → Integrations` 中，可以关联你的 Git 托管账号：

| 平台 | 认证方式 | 关联后功能 |
|------|----------|------------|
| GitHub | OAuth / Token | Clone、PR、Issue 集成、Fork |
| GitLab | OAuth / Token | Clone、MR、Issue 集成 |
| Bitbucket | OAuth | Clone、PR、Issue 集成 |
| Azure DevOps | Token | Clone、PR 集成 |

---

## 3. 界面总览

GitKraken 的界面分为三大核心区域：**左侧面板**、**中央提交图**、**右侧提交面板**。

### 3.1 左侧面板（Left Panel）

左侧面板是导航和管理的核心区域，包含以下可折叠分组：

| 分组 | 功能说明 |
|------|----------|
| **Repository** | 显示当前仓库名称，点击可查看仓库详情、配置 .gitignore 等 |
| **Branches** | 列出所有本地和远程分支，双击可切换检出，右键提供删除、重命名等操作 |
| **Remotes** | 管理远程仓库源（origin、upstream 等），可添加/删除远程地址 |
| **Tags** | 列出所有标签，右键可创建、删除、推送标签 |
| **Stashes** | 管理 Git Stash 储藏记录，可应用、删除、创建储藏 |
| **Pull Requests** | （需关联平台）显示和管理 PR/MR 列表 |

### 3.2 中央提交图（Commit Graph）

提交图是 GitKraken 的灵魂所在，它以时间轴 + DAG 有向无环图的形式可视化整个仓库的提交历史。

- **节点**：每个圆点代表一次提交，颜色对应所属分支
- **连线**：彩色线条表示分支的创建、合并和分叉路径
- **菱形**：合并提交以菱形节点标识
- **// WIP 节点**：工作区有未提交的修改时，会在分支顶部显示虚线圆圈
- **悬停**：鼠标悬停在提交节点上可查看作者、时间、SHA-1 哈希、变更文件
- **右键**：右键点击提交节点弹出操作菜单（检出、回退、创建分支、Cherry-pick 等）
- **拖拽**：可将分支拖拽到另一个分支上执行合并/变基操作

### 3.3 右侧提交面板（Commit Panel）

右侧面板在选中 `// WIP` 节点或某个提交时显示，包含：

- **Unstaged Files**：工作区已修改但未暂存的文件列表
- **Staged Files**：已暂存待提交的文件列表
- **Diff Viewer**：点击文件名展开行级差异对比，支持语法高亮
- **Commit Message**：提交信息输入框，支持多行
- **Commit 按钮**：执行提交

### 3.4 顶部工具栏

| 按钮 | 功能 | 对应 Git 命令 |
|------|------|--------------|
| 🔄 Fetch | 拉取远程更新（不合并） | `git fetch` |
| ⬇ Pull | 拉取并合并远程更新 | `git pull` |
| ⬆ Push | 推送本地提交到远程 | `git push` |
| 🌿 Branch | 创建新分支 | `git branch` |
| 📦 Stash | 储藏当前修改 | `git stash` |
| 🔄 Apply Stash | 应用储藏 | `git stash apply` |
| ⚡ Terminal | 打开内置终端 | — |

---

## 4. 仓库管理

### 4.1 克隆远程仓库

1. **打开克隆面板**：点击顶部 `File → Clone Repo`，或首页点击 `Clone a Repo`
2. **选择来源**：
   - 方式一：选择已关联的平台（GitHub/GitLab/Bitbucket），浏览选择仓库
   - 方式二：直接输入 URL（HTTPS 或 SSH）
3. **设置本地路径**：在 `Browse` 中选择本地存放目录
4. **点击 Clone**：克隆完成后点击 `Open Now` 打开仓库

```bash
# 对应的 Git 命令
git clone https://github.com/user/repo.git /path/to/local
```

### 4.2 打开本地仓库

- **打开已有仓库**：点击 `File → Open Repo`，选择本地包含 `.git` 目录的文件夹
- **初始化新仓库**：点击 `File → Init Repo`，选择目录并初始化。可选择是否添加 `.gitignore` 模板和 [[LICENSE]] 文件
- **最近打开**：首页会显示最近打开的仓库列表，点击即可快速打开

### 4.3 添加远程仓库

在左侧面板 `Remotes` 旁点击 `+` 号可添加新的远程源：

```bash
# 对应的 Git 命令
git remote add upstream https://github.com/original/repo.git
```

> **💡 Fork 工作流常用配置**
>
> Fork 开源项目时，通常 `origin` 指向你的 Fork 仓库，`upstream` 指向原项目仓库。这样可以方便地从 upstream 拉取最新代码并推送到自己的 origin。

---

## 5. 提交与暂存

### 5.1 查看文件变更

当你在工作目录中修改了文件后，GitKraken 会自动检测到变更，并在提交图顶部显示 `// WIP` 节点，同时在右上角显示 `"N file changes"` 提示。

点击 `// WIP` 节点，右侧面板会展开显示所有变更文件。

### 5.2 暂存文件（Stage）

GitKraken 提供了精细化的暂存控制：

| 操作 | 说明 | 操作方式 |
|------|------|----------|
| 暂存全部 | 将所有修改加入暂存区 | 点击 `Stage all changes` 按钮 |
| 暂存单个文件 | 将指定文件加入暂存区 | 点击文件名右侧的 `Stage file` 按钮 |
| 暂存代码块（Hunk） | 将文件中的部分修改加入暂存区 | 展开文件 Diff，点击代码块右侧的 `+` 按钮 |
| 暂存单行 | 将某一行修改加入暂存区 | 展开文件 Diff，点击行号右侧的 `+` 按钮 |
| 撤销暂存 | 将已暂存的文件移回未暂存 | 点击 Staged 区文件右侧的 `Unstage` 按钮 |
| 丢弃修改 | 放弃文件的修改（不可恢复） | 点击文件右侧的 `Discard` 按钮 |

> **⚠️ Discard 操作不可逆**
>
> `Discard` 会永久丢弃文件的修改，无法通过任何方式恢复。使用前请确认你确实不需要这些更改。

### 5.3 提交（Commit）

1. **暂存文件**：先将需要提交的文件 Stage 到暂存区
2. **输入提交信息**：在右侧 `Commit Message` 输入框中填写提交摘要（第一行）和详细描述（空行后）
3. **执行提交**：点击 `Commit` 按钮，或使用快捷键 `Ctrl+Enter`（macOS: `Cmd+Enter`）

```bash
# 对应的 Git 命令
git add .
git commit -m "提交摘要" -m "详细描述"
```

### 5.4 修改上次提交（Amend）

如果提交后发现遗漏了文件或提交信息写错了，可以修改最近一次提交：

1. **勾选 Amend**：在提交面板底部勾选 `Amend previous commit` 复选框
2. **暂存新文件（可选）**：如果有遗漏的文件，先将其 Stage
3. **修改提交信息**：在 Commit Message 中修改提交信息
4. **提交**：点击 Commit 按钮，GitKraken 会用新提交替换上次提交

> **⚠️ Amend 注意事项**
>
> 如果上次的提交已经 Push 到远程，Amend 后需要 `Force Push`。这会覆盖远程历史，可能影响其他协作者。仅在本地未推送的提交上使用 Amend。

### 5.5 提交信息规范建议

```
<type>(<scope>): <subject>

<body>

<footer>
```

| type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档变更 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构（既不是新功能也不是修 Bug） |
| `test` | 测试相关 |
| `chore` | 构建/工具/依赖变更 |

---

## 6. 分支管理

### 6.1 创建分支

GitKraken 提供多种创建分支的方式：

- **方式一**：点击顶部工具栏的 `Branch` 按钮，输入分支名后按 `Enter`
- **方式二**：在提交图中右键点击某个提交节点，选择 `Create branch here`
- **方式三**：在左侧 Branches 面板右键，选择 `Create branch`

```bash
# 对应的 Git 命令
git branch feature/login
git checkout feature/login
# 或者合并为一条
git checkout -b feature/login
```

> **💡 从特定提交创建分支**
>
> 右键点击提交图中的任意提交节点，选择 `Create branch here`，可以基于历史中的某个时间点创建分支，非常适合用于修复旧版本的 Bug。

### 6.2 切换分支（Checkout）

- **双击分支名**：在左侧面板双击分支名即可切换
- **右键 → Checkout**：右键点击分支选择 Checkout
- **双击提交节点**：直接双击提交图中的节点，会切换到该提交所在的分支

### 6.3 重命名分支

右键点击本地分支 → `Rename` → 输入新名称

```bash
git branch -m old-name new-name
```

### 6.4 删除分支

右键点击本地分支 → `Delete`

> **⚠️ 删除分支注意**
>
> 不能删除当前所在分支。如果分支有未合并的提交，GitKraken 会提示警告。强制删除需使用 `git branch -D`。

### 6.5 设置上游分支

推送本地分支到远程时，GitKraken 会自动设置上游跟踪关系。也可以手动设置：

右键点击本地分支 → `Set Upstream` → 选择远程分支

### 6.6 Cherry-pick（拣选）

Cherry-pick 可以将某个分支上的特定提交应用到当前分支，而不需要合并整个分支：

1. **切换到目标分支**：先 Checkout 到你想要应用提交的分支
2. **右键点击源提交**：在提交图中找到要拣选的提交，右键点击
3. **选择 Cherry-pick**：点击 `Cherry-pick commit`，该提交会被应用到当前分支

```bash
git cherry-pick <commit-sha>
```

---

## 7. 合并与变基

### 7.1 合并（Merge）

将一个分支的修改合并到当前分支。GitKraken 支持直观的拖拽合并：

1. **切换到目标分支**：先 Checkout 到合并的目标分支（如 `main`）
2. **拖拽源分支**：在左侧面板或提交图中，将源分支（如 `feature/login`）拖拽到目标分支上
3. **选择 Merge**：在弹出菜单中选择 `Merge source-branch into target-branch`

也可以右键点击源分支，选择 `Merge source-branch into target-branch`。

```bash
# 先切换到目标分支
git checkout main
# 合并源分支
git merge feature/login
```

### 7.2 变基（Rebase）

变基会将当前分支的提交"移植"到目标分支的最新提交之上，使提交历史保持线性。

1. **切换到源分支**：Checkout 到需要变基的分支（如 `feature/login`）
2. **拖拽到目标分支**：将源分支拖拽到目标分支（如 `main`）上
3. **选择 Rebase**：在弹出菜单中选择 `Rebase source-branch onto target-branch`

```bash
# 先切换到源分支
git checkout feature/login
# 变基到目标分支
git rebase main
```

> **⚠️ Merge vs Rebase 选择**
>
> - **Merge**：保留完整的分支历史，适合公共分支和团队协作
> - **Rebase**：产生线性的提交历史，适合个人功能分支整理
> - **黄金法则**：不要对已推送到远程的公共分支执行 Rebase！

### 7.3 回退提交（Revert）

Revert 会创建一个新的提交来撤销之前的修改，不会改变提交历史：

右键点击提交节点 → `Revert commit`

```bash
git revert <commit-sha>
```

### 7.4 重置（Reset）

Reset 会将当前分支指针移动到指定提交，有三种模式：

| 模式 | 暂存区 | 工作区 | 说明 |
|------|--------|--------|------|
| **Soft** | 保留 | 保留 | 仅移动 HEAD 指针，修改保留在暂存区 |
| **Mixed**（默认） | 清除 | 保留 | 移动 HEAD，暂存区清空，修改保留在工作区 |
| **Hard** | 清除 | 清除 | 完全重置，所有修改丢失（不可恢复） |

操作方式：右键点击目标提交 → `Reset branch to this commit` → 选择模式。

> **🚫 Hard Reset 危险**
>
> `Hard Reset` 会永久丢弃工作区和暂存区的所有修改，且无法通过 `git reflog` 恢复被丢弃的工作区修改。务必谨慎使用。

---

## 8. 远程操作

### 8.1 Fetch（获取）

Fetch 从远程仓库下载最新信息，但不会修改本地工作区。它可以让你了解远程分支的最新状态。

- 点击顶部工具栏的 `Fetch` 按钮
- 或快捷键 `Ctrl+Shift+F`

```bash
git fetch --all
git fetch origin
```

### 8.2 Pull（拉取）

Pull = Fetch + Merge，从远程拉取并合并到当前分支。

- 点击顶部工具栏的 `Pull` 按钮
- 可选择 `Pull (fast-forward only)` 或 `Pull (no fast-forward)`

```bash
git pull origin main
# fast-forward only
git pull --ff-only origin main
# no fast-forward（总是创建合并提交）
git pull --no-ff origin main
```

### 8.3 Push（推送）

将本地提交推送到远程仓库。

- 点击顶部工具栏的 `Push` 按钮
- 首次推送新分支时，GitKraken 会提示设置上游分支

```bash
# 推送当前分支
git push origin feature/login

# 首次推送并设置上游
git push -u origin feature/login

# 强制推送（Rebase 或 Amend 后使用）
git push --force-with-lease origin feature/login
```

> **⚠️ Force Push 风险**
>
> Force Push 会覆盖远程历史，可能导致其他协作者的工作丢失。GitKraken 提供 `Force Push` 选项，使用时需格外谨慎。推荐使用 `--force-with-lease` 而非 `--force`。

### 8.4 Fast-Forward vs No Fast-Forward

| 模式 | 行为 | 提交历史 |
|------|------|----------|
| Fast-Forward | 直接移动分支指针，不创建合并提交 | 线性，无合并节点 |
| No Fast-Forward | 即使可以快进，也创建合并提交 | 保留分支拓扑，有合并节点 |

---

## 9. 冲突解决

当合并或变基时，如果两个分支修改了同一文件的同一区域，Git 无法自动合并，就会产生冲突。GitKraken 提供了强大的内置合并工具来帮助你解决冲突。

### 9.1 冲突界面

当冲突发生时，GitKraken 会自动打开合并工具，界面分为四列：

| 列 | 内容 | 说明 |
|----|------|------|
| 左列（A） | 当前分支（Local/Target）的版本 | 你当前所在分支的内容 |
| 中列（Output） | 合并结果预览 | 解决冲突后的最终文件内容 |
| 右列（B） | 源分支（Remote/Source）的版本 | 被合并分支的内容 |
| 底部 | 冲突文件列表 | 所有存在冲突的文件 |

### 9.2 解决冲突操作

对于每个冲突块，你可以：

| 操作 | 说明 |
|------|------|
| ⬅ 使用左侧（A） | 保留当前分支的修改，丢弃源分支的修改 |
| ➡ 使用右侧（B） | 保留源分支的修改，丢弃当前分支的修改 |
| ⬅➡ 同时保留两侧 | 两个分支的修改都保留，按顺序排列 |
| ✏️ 手动编辑 | 在中间列直接编辑合并结果 |
| 🗑 删除冲突块 | 完全移除冲突区域的代码 |
| ↩️ 撤销选择 | 回到冲突的初始状态，重新选择 |

### 9.3 解决流程

1. **查看冲突文件列表**：底部面板显示所有冲突文件，点击文件名查看冲突详情
2. **逐块解决冲突**：对每个冲突块选择保留方案（A / B / 两者 / 手动编辑）
3. **保存**：所有冲突解决后，点击 `Save`
4. **暂存并提交**：GitKraken 会自动将解决后的文件加入暂存区，输入提交信息并提交
5. **继续合并/变基**：如果是变基过程中的冲突，提交后点击 `Continue rebasing` 继续

### 9.4 中止合并/变基

如果冲突太复杂或你想放弃操作，可以中止：

- 中止合并：`git merge --abort`
- 中止变基：`git rebase --abort`

在 GitKraken 中，冲突界面顶部会显示 `Abort` 按钮。

---

## 10. Pull Request 管理

GitKraken 6.5.1 内置了 PR/MR 管理功能，支持在客户端直接发起、浏览、审查 Pull Request，无需切换到浏览器。

> **ℹ️ 前提条件**
>
> 使用 PR 功能前，需要在 `Preferences → Integrations` 中关联对应的 Git 托管平台（GitHub / GitLab / Bitbucket）。

### 10.1 发起 PR

1. **推送分支**：先将你的功能分支 Push 到远程仓库
2. **点击 PR 按钮**：在顶部工具栏点击 `Pull Request` 按钮，或左侧面板 `Pull Requests` 旁的 `+` 号
3. **填写 PR 信息**：
   - 选择目标分支（Target：如 `main`）和源分支（Source：你的功能分支）
   - 填写 PR 标题和描述
4. **提交**：点击 `Create Pull Request`

### 10.2 查看 PR

在左侧面板 `Pull Requests` 分组中，可以查看所有 PR 的状态：

- **Open**：待处理的 PR
- **Merged**：已合并的 PR
- **Closed**：已关闭的 PR

### 10.3 审查 PR

点击 PR 条目可在右侧面板查看详情：

- 查看 PR 描述和变更文件
- 查看提交历史和 Diff
- 添加评论
- Approve / Request Changes
- Merge PR（如果权限允许）

### 10.4 Checkout PR 分支

在 PR 列表中右键点击某个 PR，选择 `Checkout PR branch`，可以将 PR 的分支检出到本地进行测试和验证。

---

## 11. Stash 储藏

当你正在某个分支上工作，但需要临时切换到其他分支处理紧急问题时，可以使用 Stash 将当前未完成的修改暂时储藏起来。

### 11.1 创建 Stash

1. **点击 Stash 按钮**：点击顶部工具栏的 `Stash` 按钮，或右键 `Stashes` → `Stash changes`
2. **输入名称（可选）**：可以为 Stash 命名以便后续识别，默认名称为分支名 + 提交信息
3. **确认**：工作区的修改会被储藏，工作区恢复到 HEAD 状态

```bash
git stash save "WIP: login feature"
```

### 11.2 应用 Stash

- **Apply Stash**：应用储藏但保留储藏记录
- **Pop Stash**：应用储藏并删除储藏记录

操作方式：在左侧 `Stashes` 面板中右键点击 Stash 条目，选择 Apply 或 Pop。

```bash
# 应用（保留记录）
git stash apply stash@{0}

# 弹出（删除记录）
git stash pop stash@{0}
```

### 11.3 删除 Stash

右键点击 Stash → `Delete stash`

```bash
git stash drop stash@{0}
```

### 11.4 Stash 包含未跟踪文件

默认情况下，Stash 只包含已跟踪文件的修改。如果需要包含未跟踪的新文件：

```bash
git stash save -u "include untracked"
```

---

## 12. SSH 密钥配置

使用 SSH 方式连接远程仓库可以免去每次输入密码的麻烦。GitKraken 6.5.1 内置了 SSH 密钥管理功能。

### 12.1 生成 SSH 密钥

1. **打开 SSH 设置**：`File → Preferences → Authentication → SSH`
2. **选择 Generate**：点击 `Generate new key and add to SSH agent`
3. **填写信息**：输入密钥名称和 passphrase（可选），选择密钥类型（推荐 ED25519 或 RSA 4096）
4. **添加到平台**：点击 `Copy Public Key`，然后粘贴到 GitHub/GitLab 的 SSH Keys 设置中

也可以使用命令行生成：

```bash
# 生成 ED25519 密钥（推荐）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 生成 RSA 4096 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 密钥默认存储在
# Windows: C:\Users\用户名\.ssh\id_ed25519
# macOS/Linux: ~/.ssh/id_ed25519
```

### 12.2 在 GitKraken 中配置 SSH

1. **打开 SSH 配置**：`Preferences → Authentication → SSH`
2. **选择 SSH 私钥**：点击 `Browse`，选择 `~/.ssh/id_ed25519`（或 `id_rsa`）
3. **选择 SSH Agent**：
   - **GitKraken**：使用内置 SSH Agent（推荐）
   - **System**：使用系统 SSH Agent
4. **测试连接**：Clone 一个 SSH URL 的仓库测试是否配置成功

### 12.3 SSH vs HTTPS

| 对比项 | SSH | HTTPS |
|--------|-----|-------|
| 认证方式 | 密钥对 | 用户名 + 密码/Token |
| 是否需要每次输入 | 否（密钥自动认证） | 否（凭据管理器缓存） |
| 防火墙兼容性 | 较差（需开放 22 端口） | 好（使用 443 端口） |
| 安全性 | 高（密钥不可窃取） | 中（依赖密码强度） |
| 推荐场景 | 日常开发 | 企业网络/受限环境 |

---

## 13. GitFlow 工作流

GitFlow 是一种广泛使用的 Git 分支管理模型，GitKraken 内置了对 GitFlow 的原生支持。

### 13.1 GitFlow 分支模型

```
main:    ●───────────────────────────────────────────────●──→ v1.0
          \                              \             /  
           \                              \           /   
develop:     ●──────────●──────●───────────●─────●───●──→  
                         \         \        \   /          
                          \         \        \ /           
feature/login:             ●─────●   \        ●           
                                        \      \          
release/1.0:                        ●───●────●            
                                               \          
hotfix/1.0.1:                                    ●───●    
```

| 分支类型 | 命名规范 | 来源分支 | 合并目标 | 用途 |
|----------|----------|----------|----------|------|
| `main` | main / master | — | — | 生产环境代码，只接受 release 和 hotfix 合并 |
| `develop` | develop | main | release | 日常开发集成分支 |
| `feature` | feature/* | develop | develop | 新功能开发 |
| `release` | release/* | develop | main + develop | 发布准备（版本号、文档等） |
| `hotfix` | hotfix/* | main | main + develop | 生产环境紧急修复 |

### 13.2 在 GitKraken 中初始化 GitFlow

1. **打开 GitFlow 配置**：点击左侧面板的 `Git Flow` 按钮，或 `File → Preferences → Git Flow`
2. **配置分支名称**：设置 main、develop 分支名，以及 feature、release、hotfix 的前缀（通常保持默认）
3. **点击 Init**：GitKraken 会自动创建 develop 分支并设置 GitFlow 配置

### 13.3 使用 GitFlow 操作

初始化后，左侧面板的 `Git Flow` 按钮会提供以下快捷操作：

- **New Feature**：创建新的 feature 分支（自动从 develop 创建）
- **Finish Feature**：完成 feature（自动合并回 develop）
- **New Release**：创建 release 分支
- **Finish Release**：完成 release（合并到 main + develop，打标签）
- **New Hotfix**：创建 hotfix 分支
- **Finish Hotfix**：完成 hotfix（合并到 main + develop，打标签）

---

## 14. 快捷键速查

### 14.1 通用快捷键

| 快捷键 (Win/Linux) | 快捷键 (macOS) | 功能 |
|---------------------|-----------------|------|
| `Ctrl+Enter` | `Cmd+Enter` | 提交（Commit） |
| `Ctrl+Shift+S` | `Cmd+Shift+S` | 暂存全部（Stage All） |
| `Ctrl+Shift+F` | `Cmd+Shift+F` | 获取（Fetch） |
| `Ctrl+1` | `Cmd+1` | 打开仓库 |
| `Ctrl+2` | `Cmd+2` | 克隆仓库 |
| `Ctrl+3` | `Cmd+3` | 初始化仓库 |
| `Ctrl+N` | `Cmd+N` | 新标签页 |
| `Ctrl+W` | `Cmd+W` | 关闭标签页 |
| `Ctrl+,` | `Cmd+,` | 打开偏好设置 |
| `Ctrl+F` | `Cmd+F` | 搜索提交 |
| `Esc` | `Esc` | 取消/关闭面板 |

### 14.2 分支与提交操作

| 操作 | 快捷键 | 功能 |
|------|--------|------|
| `Ctrl+B` | 创建分支 | — |
| 双击分支名 | — | 切换分支 |
| 双击提交节点 | — | 切换到该提交所在分支 |
| 右键提交 → Cherry-pick | — | 拣选提交 |
| 右键提交 → Revert | — | 回退提交 |
| 右键提交 → Reset | — | 重置到此提交 |
| 拖拽分支到分支 | — | 合并/变基 |

### 14.3 文件与 Diff 操作

| 操作 | 功能 |
|------|------|
| 点击文件名 | 展开/折叠 Diff 视图 |
| Diff 中点击 `+` | 暂存该行/块 |
| Diff 中点击 `−` | 撤销暂存该行/块 |
| 点击 `Edit` | 在外部编辑器中打开文件 |
| 右键文件 → View History | 查看文件历史 |
| 右键文件 → Blame | 查看文件每行的最后修改者 |
| 右键文件 → Open in File Manager | 在文件管理器中打开 |

---

## 15. 常见问题（FAQ）

### Q1: GitKraken 6.5.1 还能免费打开私有仓库吗？

**可以。** 6.5.1 是最后一个免费支持私有仓库的版本。从 6.5.2 开始，免费版仅支持公开仓库，私有仓库需要 Pro 许可证。如果你需要免费管理私有仓库，请保持使用 6.5.1 版本。

### Q2: 如何关闭自动更新？

GitKraken 6.5.1 默认会检查更新。如果不想升级到收费版本，可以在 `Preferences → General` 中关闭自动更新。如果已被提示更新，请点击 `Skip` 或 `Remind me later`。

### Q3: 提交图太慢 / 大仓库卡顿怎么办？

- 在 `Preferences → General` 中减少 `Max commits to load`
- 关闭 `Show remote branches` 以减少渲染节点
- 使用 `Search / Filter` 过滤特定分支
- 对超大仓库考虑使用浅克隆 `git clone --depth 1`

### Q4: Push 时提示 "Permission denied" 怎么办？

通常是 SSH 密钥未正确配置。检查步骤：

1. 确认 `~/.ssh/id_ed25519`（或 `id_rsa`）存在且有读权限
2. 确认公钥已添加到 GitHub/GitLab 的 SSH Keys 设置
3. 在 GitKraken `Preferences → SSH` 中确认私钥路径正确
4. 在终端执行 `ssh -T git@github.com` 测试连接

### Q5: 如何查看文件的历史修改记录（File History）？

在提交面板中右键点击文件 → `View File History`，GitKraken 会在提交图中仅显示涉及该文件的提交，方便追踪文件的变更历史。

### Q6: 如何查看每行代码是谁写的（Blame）？

右键点击文件 → `Blame`，GitKraken 会打开 Blame 视图，显示每行代码的最后修改者和提交信息。

### Q7: 如何配置 .gitignore？

- **方式一**：在 GitKraken 中右键未跟踪文件 → `Ignore`（可选择仅忽略此文件或同类文件）
- **方式二**：直接编辑项目根目录的 `.gitignore` 文件
- **方式三**：在 `File → .gitignore` 中编辑（如仓库已关联）

### Q8: 如何撤销已经 Push 的提交？

推荐使用 `Revert`（右键提交 → Revert commit），它会创建一个新的提交来撤销之前的修改，不会破坏提交历史。不推荐使用 Reset + Force Push，因为会影响其他协作者。

### Q9: GitKraken 6.5.1 支持 Gitee（码云）吗？

GitKraken 官方集成的平台是 GitHub、GitLab、Bitbucket 和 Azure DevOps。对于 Gitee，可以通过 URL 方式 Clone（HTTPS 或 SSH），PR 管理功能不支持，但所有 Git 基本操作（提交、推送、拉取、合并等）完全可用。

### Q10: 如何在多台电脑间同步 GitKraken 配置？

登录同一个 GitKraken 账号，部分偏好设置和关联的平台账号会自动同步。但 SSH 密钥、本地路径等需要手动配置。

---

> 🦑 **GitKraken 6.5.1 使用手册** · 生成于 2026-07-13
>
> 本手册基于 GitKraken 6.5.1 版本编写，供学习和参考使用。
