# 研究目标与问题定义（v0.1）

> 本文档回答三个问题：我们要做什么、为什么值得做、怎么算做成。建议在启动期第一、二次例会上讨论定稿。

## 一、项目定义

将 **OpenClaw**（开源自主 AI 助手框架）的 **Skill** 体系，**移植/适配**到 **HarmonyOS（鸿蒙）**上的大模型运行环境，形成可复用、可推广的"跨 OS 大模型技能迁移"方法论与原型。

## 二、OpenClaw Skill 关键机制（移植对象）

- **Skill 物理载体**：一个目录 + `SKILL.md`（YAML frontmatter + Markdown 正文），遵循 [AgentSkills](https://agentskills.io) 规范。
- **frontmatter 关键字段**：`name`、`description`、`homepage`、`user-invocable`、`disable-model-invocation`、`command-dispatch` 等。
- **门控（Gating）**：`metadata.openclaw.requires.bins|anyBins|env|config`、`os` 平台筛选（`darwin|linux|win32`）、`always` 强制启用。
- **加载与优先级**：工作区 > 项目智能体 > 个人智能体 > 托管 > 内置 > 额外目录（同名覆盖）。
- **运行期行为**：环境变量注入、系统提示词注入（XML 快照）、会话级快照、Skill Workshop 提案、ClawHub 安装/验证。
- **依赖**：Skill 常依赖宿主二进制（`bins`）、外部命令、CLI 工具、API Key 等，这些在鸿蒙侧多数不存在，是做移植/抽象时的核心难点。

## 三、问题与挑战

1. **平台依赖**：`os` 字段与 `bins` 依赖面向 darwin/linux/win32，鸿蒙不属于其中。
2. **工具调用抽象**：Skill 通过提示词驱动模型调用工具；鸿蒙大模型（盘古等）的工具调用协议 / Function Calling 与 OpenClaw 的 CLI 工具协议不同。
3. **运行时宿主**：OpenClaw 依赖 Node.js 与宿主 shell；鸿蒙端无 linux shell 环境（HarmonyOS NEXT 为微内核 + ArkTS/Native 运行时）。
4. **安全与沙箱**：Skill 被视为不受信代码，需要沙箱隔离；鸿蒙侧缺少成熟的进程沙箱/权限体系对接。
5. **迁移规模**：Skill 数量多、依赖杂，逐一手工改成本高——这正是方向 B（Skill VM）要解决的扩展性问题。

## 四、研究问题（RQ，可进一步拆分）

- **RQ1**：OpenClaw Skill 到鸿蒙大模型的可移植性边界在哪？哪些能直接复用/需要适配/无法迁移？
- **RQ2**：如何设计一种与 OS 解耦的 Skill 中间表示与运行时（Skill VM），使同一 Skill 能在多种宿主（含鸿蒙）上运行？
- **RQ3**：两种路径（手工移植 vs Skill VM）在移植成本、运行开销、功能完整度、维护性上的量化差异如何？

## 五、验收标准（初步）

- 方向A：在鸿蒙/鸿蒙模拟环境上成功运行 ≥5 个代表性 Skill，并有移植档案与评测数据。
- 方向B：给出 Skill VM 的 IR 规范 + 最小可运行原型，至少在 OpenClaw 宿主与鸿蒙宿主两个目标上各跑通 1 个示例 Skill。
- 两方向：输出一篇完整文献综述 + 对比评测报告，作为论文/项目报告素材。

## 六、名词表

| 名词 | 说明 |
| --- | --- |
| Skill | OpenClaw/AgentSkills 规范下的技能，Markdown 指令集 + 工具绑定 |
| SKILL.md | 技能的入口文件，YAML frontmatter + Markdown 正文 |
| HarmonyOS / HarmonyOS NEXT | 华为操作系统；NEXT 版本无 Android 兼容层 |
| Skill VM | 本研究提出的中间表示 + 运行时抽象层（方向 B） |
| 移植档案 | 每个 Skill 一次移植过程的完整记录（模板见 `templates/`） |
