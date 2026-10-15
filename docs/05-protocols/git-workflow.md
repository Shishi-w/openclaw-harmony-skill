# Git 工作流与协作规范

> 统一团队的提交、分支、PR、命名约定，保证公开仓库整洁、历史清晰、可回溯。

## 一、分支模型（简化 GitHub Flow）

只用一个长期分支 `main`（或 `master`），日常开发用**功能分支**，通过 **Pull Request** 合并。

```
main  ←——（merge）——  feature/方向组/#issue-标题
```

分支命名规范：

```
feature/<方向A|方向B|doc|env>/<短描述>
例：
  feature/A/port-file-reader
  feature/B/vm-ir-schema
  feature/doc/roadmap-v2
  feature/env/ci-actions
```

> 已发布的"已完成"里程碑可以打 tag：`v0.1`、`v0.2`。

## 二、Commit Message 规范（Conventional Commits 精简版）

```
<type>(<scope>): <subject>
```

- **type**：`feat`（新功能/新 Skill）、`fix`（修复）、`docs`（文档）、`refactor`（重构）、`test`（测试）、`chore`（杂项/构建）、`style`（格式）。
- **scope**（可选）：模块/方向，如 `A`、`B`、`env`、`doc`。
- **subject**：简短说明，用中文或英文均可，**一句话说清做了什么**。

示例：

```
feat(A): 移植 file-reader Skill 到鸿蒙宿主
docs: 更新 README 目录导航
ci(env): 添加 GitHub Actions 基础检查
```

## 三、完整工作流（组员日常）

1. **拉取最新**
   ```bash
   git pull --rebase
   ```
2. **建分支**
   ```bash
   git checkout -b feature/A/port-file-reader
   ```
3. **开发 + 提交**（小步提交，逻辑完整再 commit）
   ```bash
   git add .
   git commit -m "feat(A): 新增 file-reader 移植档案与改写后的 SKILL.md"
   ```
4. **推送分支**
   ```bash
   git push -u origin feature/A/port-file-reader
   ```
5. **发起 PR**：GitHub 网页上创建 Pull Request，写明改动与自测结果，@相关负责人 review。
6. **通过后合并**到 `main`，删除本地分支。

## 四、Code Review 原则

- 每个 PR 至少 1 名负责人 review 通过后合并。
- 关注点：功能正确性、是否引入密钥、是否符合命名/文档规范、可复现性（能否跑通）。
- 用 GitHub PR 内评论讨论，**不要在私聊里改来改去**，保证留痕。

## 五、文件/目录命名约定

- 文档/目录：全小写 + 连字符（如 `git-intro.md`、`division-of-labor.md`）。
- Skill 目录：`SKILL.md`，文件夹名用业务英文（如 `file-reader`）。
- 镜像文件：参考真实文件用 `xxx.example`（如 `.env.example`），避免提交真实密钥。

## 六、禁止事项

- ❌ 直接 push 到 `main`（除非紧急修复且获授权）。
- ❌ 提交任何密钥/账号/私密文件（仓库已配置 `.gitignore`，但请自觉检查 `git status`）。
- ❌ 把本人或他人隐私（手机号/身份证等）写进公开仓库。
- ❌ 用 `git push --force` 覆盖他人提交（如确需，先沟通 + 未推分支才可 force）。

## 七、新人第一课回顾

- 如果对以上不熟，先回头过一遍 `docs/03-environment/git-intro.md`。
- 每周例会可花 5 分钟由老手带新手做一次"建分支→PR→合并"实操。

## 八、.gitignore 说明

仓库已内置 `.gitignore`，忽略：系统文件、IDE 配置、构建产物、`.env`/密钥、OpenClaw 运行产物（`.openclaw/`）、鸿蒙构建产物（`.ohos/`、`.hap`）等。新增大类型文件时优先更新 `.gitignore` 而不是硬提交。
