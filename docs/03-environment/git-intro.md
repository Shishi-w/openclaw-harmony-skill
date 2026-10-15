# 环境搭建指南（Git 基础）

> 面向组内新手成员，一步步从零配置。老手可跳过，直接看"进阶"小节。

## 0. 前置：需要一个 GitHub 账号

1. 打开 https://github.com 注册（邮箱 + 用户名）。
2. 用户名请用"便于识别的英文/拼音"，例如 `zhang-san`。
3. 把用户名发给组长，由组长邀请你加入仓库协作者。

## 1. 安装 Git（Windows / macOS / Linux）

- **Windows**：下载 https://git-scm.com/download/win 安装包，一路 Next（默认即可，注意勾选"Add to PATH"）。
- **macOS**：`brew install git`，或直接安装 Xcode Command Line Tools。
- **Linux(Ubuntu/Debian)**：`sudo apt install git`。
- 验证：打开终端（Windows 用 PowerShell 或 Git Bash），输入

```bash
git --version
```

出现 `git version 2.x.x` 即成功。

## 2. 配置身份（一次即可）

```bash
git config --global user.name "你的名字"        # 例如 Zhang San
git config --global user.email "你的邮箱"       # 与 GitHub 注册邮箱一致
```

## 3. 配置 SSH（推荐，免密推送）

```bash
ssh-keygen -t ed25519 -C "你的邮箱"   # 一路回车即可
```

- Windows：默认生成在 `C:\Users\你的用户名\.ssh\id_ed25519.pub`
- macOS/Linux：`~/.ssh/id_ed25519.pub`

复制公钥内容，到 GitHub → Settings → SSH and GPG keys → New SSH key 粘贴保存。

验证：

```bash
ssh -T git@github.com
```

看到 `Hi <用户名>! You've successfully authenticated` 即成功。

## 4. 克隆仓库

```bash
git clone git@github.com:<组织名>/openclaw-harmony-skill.git
cd openclaw-harmony-skill
```

> 如果组织名还没有，先由组长在 GitHub 上创建组织并把仓库迁入（见 README 说明）。

## 5. 日常使用三板斧

```bash
git pull            # 拉取最新
# ... 修改/新增文件 ...
git add .           # 暂存改动
git commit -m "feat: 说明你做了什么"   # 提交
git push            # 推送到远程
```

> 更规范的分支/PR 流程见 `docs/05-protocols/git-workflow.md`，新手先掌握"拉-改-提-推"即可。

## 6. 常用检查命令

```bash
git status          # 当前工作区状态
git log --oneline   # 提交历史
git diff            # 查看未暂存改动
```

## 7. 遇到问题（FAQ）

| 问题 | 处理 |
| --- | --- |
| push 被拒绝（remote rejected） | 先 `git pull --rebase` 再 `git push` |
| 提示权限不足 | 检查 SSH key 是否已添加到 GitHub |
| 中文乱码 | 终端编码设为 UTF-8 |
| 提交者信息不对 | 检查第 2 步 `git config --global` |

## 8. 进阶（可选，了解即可）

- **分支**：`git checkout -b 分支名`，改完再合并。
- **PR**：GitHub 网页上发起 Pull Request，让组长/负责人 review。
- **GitHub Desktop**：图形化 GUI，适合不熟命令行的成员（下载 https://desktop.github.com）。

> 所有新人完成本指南后，在群里报告"环境 OK"，并在 `team/member-roster.md` 登记。
