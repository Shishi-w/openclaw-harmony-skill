# 方法论文档：方向 B —— Skill VM（技能虚拟机，创新路径）

> 目标：设计并实现"把 Skill 与具体操作系统/宿主解耦"的抽象层，使同一 Skill 一次编写、多端运行。这是本项目的创新点与论文核心贡献。

## 一、核心思想

类比 JVM / LLVM / WebAssembly：

- **一次编写，到处运行**：Skill 不再面向特定 OS，而是面向一个中立的"Skill 中间表示（IR）与运行时（Skill VM）"。
- 宿主（Host）：提供统一的能力接口，如工具调用、环境探测、文件/网络、权限沙箱。
- 适配器（Adapter）：把同一套 VM API 映射到具体宿主——OpenClaw（Node/CLI）是一个宿主，鸿蒙（ArkTS/NAPI）是另一个宿主。

```
Skill (SKILL.md + 工具声明)
        │  编译/解析
        ▼
  Skill IR（中立 Schema，与 OS 无关）
        │
        ▼
   Skill VM Runtime（门控解析、上下文注入、调度、沙箱）
        │               │
        ▼               ▼
  Host Adapter A       Host Adapter B ...
  (OpenClaw/通用)      (HarmonyOS)
```

## 二、三大子模块（对应 B1/B2/B3 分工）

### B1：Skill 中间表示（IR）设计
- 将 `SKILL.md` 的 frontmatter 与正文结构化，抽出：能力声明（tools、deps）、门控条件（os、bins、env、config）、指令逻辑（Markdown 正文）。
- 制定 IR Schema（JSON/YAML），与具体宿主的路径/命令解耦。参考 AgentSkills 规范，但引入"抽象工具类型"（如 `file.read`、`http.get`）而不绑定具体实现。
- 产出：IR 规范文档 + Schema 定义文件（`code/skill-vm/schema/`）。

### B2：Skill VM 运行时（Runtime）
- 门控引擎：复刻/抽象 OpenClaw 的门控逻辑（`requires.*`、`os`），但要与 OS 无关 → 用"能力探测接口"代替硬编码平台判断。
- 上下文注入与提示词构建：把 Skill 渲染成宿主模型可理解的提示词（参考 OpenClaw 的 XML 快照机制，做成可插拔渲染器）。
- 调度与沙箱：定义执行权限模型，抽象出设备上的资源（文件、网络、命令受限执行）。
- 产出：最小可运行 VM 原型（`code/skill-vm/`），在 PC 上的"模拟宿主"与 OpenClaw 宿主各跑通至少 1 个 Skill。

### B3：宿主适配层（Adapter）
- 定义宿主抽象接口（Host Interface）：`detectCapability()`、`openFile()`、`httpRequest()`、`runIsolated()` 等。
- 实现 OpenClaw/通用宿主适配器（用 Node 验证逻辑）。
- 实现 **HarmonyOS 宿主适配器**（ArkTS + NAPI），对接 `@ohos.file.fs`、`@ohos.net.http`、意图框架等 → 这是落地到鸿蒙的关键。
- 产出：Adapter 接口文档 + 鸿蒙适配器代码 + 可行性报告。

## 三、设计要点与难点

1. **能力解耦**：Skill 只能声明"抽象能力"（我要读文件、要联网），不能硬编码"我调 `cat` 命令"——由 VM 映射到当前宿主。
2. **门控中立化**：把 `os: linux` 这类平台门控，改为"能力门控"（`requires.capabilities: [file.write]`）。
3. **安全**：Skill 是不受信内容，VM 要提供权限边界与沙箱，鸿蒙侧权限模型对接（这也是论文的一个点）。
4. **性能**：VM 抽象层可能带来开销 → 需要评测（与方向 A 直接实现对比），见 `comparison.md`。
5. **模型协议适配**：不同宿主的大模型 Function Calling 协议不同 → VM 渲染出的"Skill 调用描述"要能转成各宿主协议。

## 四、里程碑路径

| 阶段 | 内容 | 可验证产出 |
| --- | --- | --- |
| M1 规范 | 完成 IR Schema + Host Interface 文档 | 评审通过 |
| M2 核心 | VM 运行时在 PC/OpenClaw 宿主上跑通 1 个示例 Skill | 命令行演示 |
| M3 鸿蒙 | HarmonyOS 适配器跑通同一 Skill | 模拟器/真机演示 |
| M4 评测 | 多 Skill 对比评测 + 论文实验 | 评测报告 |

## 五、给方向 A 的接口

方向 A 的移植档案对 VM 组件进行"差分"：每个 Skill 里哪些是共性部分（进 IR/VM）、哪些是宿主特例（进 Adapter）。B 组据此迭代 IR 设计。

## 六、潜在论文定位（供选题参考）

- "面向大模型技能的跨平台运行时：Skill VM 的设计与实现"
- 类似 WASM 之于 Web，Skill VM 之于 Agent Skill：一次编写、多端（含 HarmonyOS）运行。
- 贡献点：IR 设计、门控中立化、宿主抽象、鸿蒙适配、与手工移植的量化对比。
