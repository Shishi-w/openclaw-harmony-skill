# OpenClaw Skill → HarmonyOS 大模型移植研究

> 南京大学软件工程学习小组 · 研究项目仓库
> 研究目标：将开源智能体框架 **OpenClaw** 的 Skill 体系移植 / 适配到 **HarmonyOS（鸿蒙）** 上的大模型（LLM）运行时，形成可复用、可推广的迁移方法。

本仓库是团队共享的**代码管理、文档存档与协作枢纽**，组织所有研究过程中产生的环境搭建方法、论文笔记、方法复现、代码实现与阶段性成果。

---

## 一、项目背景

- **OpenClaw**：一个开源自主 AI 助手框架（原名 Clawdbot），运行在本机或云端，通过"Skill（技能）"赋予智能体调用工具、完成任务的能力。
- **OpenClaw Skill 机制要点**：每个 Skill 是一个目录，内含 `SKILL.md`（YAML frontmatter + Markdown 正文），遵循 [AgentSkills](https://agentskills.io) 规范；支持依赖门控（`requires.bins/env/config`）、环境变量注入、加载优先级、Skill Workshop、ClawHub 注册表等。
- **HarmonyOS + 大模型**：华为鸿蒙系统（含 HarmonyOS NEXT / ArkTS / Native 运行时），结合盘古等大模型能力，是目标落地平台。
- **核心问题**：OpenClaw Skill 依赖 Markdown 指令、宿主工具调用、平台感知（`os: darwin/linux/win32`）、外部二进制等，其设计面向传统 OS，无法直接在鸿蒙上运行。如何高效、系统化地将 Skill 能力迁移到鸿蒙大模型生态，是本研究要回答的问题。

## 二、两个研究方向

| 方向 | 名称 | 思路 | 定位 |
| --- | --- | --- | --- |
| **A** | 逐 Skill 移植（手工适配） | 逐个 Skill 分析依赖与工具调用，针对性改写 frontmatter / 正文 / 工具绑定，使其适配鸿蒙运行环境 | 传统、渐进、工程化，能快速产出可用成果 |
| **B** | Skill VM（技能虚拟机） | 设计 Skill 的中间表示 / 抽象运行时 / 沙箱与宿主适配层，使 Skill 与具体 OS 解耦，一次编写、多端运行 | 创新、前瞻，面向论文发表与长期价值 |

> 两方向**并行推进、相互支撑**：方向 B 为方向 A 提供架构抽象，方向 A 为方向 B 提供真实迁移案例与验证数据。详细方法论见 `docs/04-methodology/`。

## 三、仓库结构

```
.
├── README.md                      # 本文件：项目总览与导航
├── docs/
│   ├── 01-overview/              # 研究目标、问题定义、路线图
│   ├── 02-research/              # 论文研究：检索方法、综述、阅读笔记
│   ├── 03-environment/           # 环境搭建：Git/OpenClaw/HarmonyOS/大模型后端
│   ├── 04-methodology/           # 方法论文档：方向A(逐Skill移植)、方向B(Skill VM)
│   └── 05-protocols/             # 协作规范：Git 工作流、会议模板、任务看板
├── team/
│   └── division-of-labor.md      # 分工表与小组划分
├── code/
│   ├── ports/                    # 方向A：已移植/迁移中的 Skills
│   └── skill-vm/                 # 方向B：Skill VM 原型代码
├── templates/                    # 各类复用模板（移植记录/会议纪要）
└── notes/meetings/               # 会议纪要存档
```

## 四、快速开始（新人入门路径）

如果你是第一次加入本小组，请按以下顺序完成：

1. **加入 GitHub**：注册账号，把用户名告诉组长，获得本仓库 collaborator 权限。
2. **环境搭建**：按 `docs/03-environment/`完成本机环境。
3. **了解领域**：通读 `docs/01-overview/research-goals.md` 与 `docs/04-methodology/` 下两个方向的介绍。
4. **领取任务**：在 `team/division-of-labor.md` 找到你所在小组，追踪 `docs/05-protocols/task-board.md` 中的任务。
5. **正式干活**：遵循 `docs/05-protocols/git-workflow.md` 的分支与 PR 规范提交贡献。

```bash
git clone https://github.com/Shishi-w/openclaw-harmony-skill.git
cd openclaw-harmony-skill
```

## 五、协作与沟通

- **代码与文档**：全部沉淀在本仓库（GitHub）。
- **任务追踪**：使用本仓库的 Issues + Projects 看板（见 `docs/05-protocols/task-board.md`）。
- **会议**：例会纪要统一存放 `notes/meetings/`，使用 `templates/meeting-template.md`。
- **论文**：每人阅读的论文笔记放在 `docs/02-research/papers/`，使用 `templates/paper-reading-template.md`。

## 六、路线图（概览）

| 阶段 | 内容 | 里程碑产出 |
| --- | --- | --- |
| 0. 启动期（第 1-2 周） | 建仓、环境搭建、分工确认 | 文档齐全、全员环境可用 |
| 1. 调研期（第 3-5 周） | 论文研读、OpenClaw Skill 机制拆解、需求分析 | 文献综述 + Skill 清单与分类 |
| 2. 验证期（第 6-10 周） | 方向A跑通 3-5 个 Skill 移植；方向B完成 VM 架构设计与最小原型 | 可运行原型 + 对比评测 |
| 3. 深化期（第 11-15 周） | 批量移植、VM 完善、评测体系建立 | 迁移工具链 + 评测报告 |
| 4. 输出期（第 16 周起） | 论文/项目报告撰写 | 论文草稿 + 最终报告 |

详见 `docs/01-overview/roadmap.md`。

## 七、开源声明

本仓库公开，遵循 MIT 协议（见 `LICENSE`）。所有内容均为团队成员原创（除非注明引用来源），发布前请确保不包含任何校内/涉密材料。

---

*本仓库由南京大学软件工程学习小组维护。*
