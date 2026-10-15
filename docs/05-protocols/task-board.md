# 任务看板使用说明

> 协调 20+ 人协作的任务追踪方案。推荐用 GitHub 内置的 **Projects（看板）** 承载，任务列表文档存于本仓库便于追溯。

## 一、推荐方案：GitHub Projects

1. 仓库 → **Projects** → New project（选 Board 看板）。
2. 建立以下列（列名可自行调整）：
   - **待办（Backlog）**：收集到的任务，未分配。
   - **进行中（In Progress）**：有人认领并开始做。
   - **待审查（In Review / PR）**：已提交 PR 等待 review。
   - **已完成（Done）**：已合并/验收。

3. 每个任务建一个 **Issue**，在 Projects 里拖动状态即可。
   - Issue 标题规范：`[方向A|B|支撑] 任务短描述`，如 `[A] 移植 file-reader Skill`。
   - Issue 正文用 `templates/issue-template.md`（见下），包含：目标、结果定义、负责人、截止、关联 PR。

## 二、认领任务的规则

1. 组长/负责人在例会上发布任务 → 成员在 Projects 中认领（Assign yourself + 拖到 In Progress）。
2. **一件事一个人主责**，避免多人重复。协作者在 Issue 里写清楚。
3. 任务完成标准明确（参考 Issue 模板的"结果定义"栏）。
4. 每周例会过一遍看板：哪些阻塞、哪些超期、哪些需要协调。

## 三、任务状态流转约定

- Backlog → In Progress：认领。
- In Progress → In Review：提交 PR 后把 PR 地址贴到 Issue。
- In Review → Done：负责人 review 合并后，关掉 Issue。

## 四、离线/文档化备份

若暂时不用 GitHub Projects，可在此建 `TASKBOARD.md` 存放当前任务清单表格（任务/负责人/状态/截止/关联PR），由 C3 文档组定期同步。保持"看板或文档二选一"为准，避免两边不一致。

## 五、常见任务类型示例

- [A] 移植 Skill：file-reader
- [B] 定义 Skill IR Schema v1
- [B] 实现 Host Interface 接口文档
- [支撑] 编写评测基准集 benchmark.md
- [支撑] 论文综述：跨平台 VM 相关研究
- [env] 配置 GitHub Actions 基础 CI
