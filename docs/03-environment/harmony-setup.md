# 环境搭建指南：HarmonyOS 开发环境

> 目标落地平台是鸿蒙（HarmonyOS），方向 A 与方向 B 的宿主适配最终都要跑在鸿蒙上。本文给出一条**低成本起步**路径，具体版本以官方为准。

## 1. 目标平台说明

- **HarmonyOS**：华为自研操作系统。
- **HarmonyOS NEXT**：自 2024 年起无 Android 兼容层的版本，应用开发使用**ArkTS（TypeScript 方言）**，运行在 **方舟编译器 / ArkTS 运行时 + OpenHarmony** 之上；原生能力可通过 **Native（C/C++）/ NAPI** 对接。
- 大模型侧：鸿蒙的 **盘古（Pangu）/ Harmony Intelligence** 智能体能力与"意图框架"（Intent Framework）——这是 Skill 移植后要对接的模型与调度能力。

## 2. 开发工具

| 工具 | 用途 | 下载/说明 |
| --- | --- | --- |
| DevEco Studio | 鸿蒙官方 IDE（基于 IntelliJ），含模拟器 | 华为开发者联盟 developer.huawei.com 下载 |
| HarmonyOS SDK | 编译鸿蒙应用 | DevEco 内置，或通过 SDK Manager |
| HVD（HarmonyOS Virtual Device）模拟器 | 无真机时的运行调试环境 | DevEco 内创建 |
| Node.js | 很多脚手架/工具链依赖 | 与 OpenClaw 环境共用 |

> 学生可使用**个人开发者账号**登录 DevEco 开发者联盟，免费创建应用签名与模拟器测试（按官方学生/个人免费政策）。

## 3. 最小起步步骤（无真机方案）

1. 安装 DevEco Studio（勾选 HarmonyOS SDK + 模拟器组件）。
2. 注册华为开发者账号并完成实名（学生可用校园身份）。
3. 新建一个 **Empty Ability** 工程（语言选 ArkTS），跑通默认"Hello World"在模拟器上。
4. 了解 ArkTS 基础语法（TypeScript + 声明式 UI 的 ArkUI）。

## 4. 研究侧需要重点打通的能力（按优先级）

方向 A / B 的鸿蒙适配说明参考 `docs/04-methodology/`，工程能力重点包括：

- **网络/HTTP**：`@ohos.net.http` 模块，`http.createHttp()` 发起请求 → 对接 LLM API 与第三方工具。
- **文件系统**：`@ohos.file.fs`，读写 Skill 需要的本地文件。
- **进程/命令执行**：鸿蒙 **没有**通用 `shell exec`，这是移植的最大障碍 → 需要评估：调用 Native/AWK、还是用受限能力替代、还是通过设备侧宿主机远程执行（研究点）。
- **ArkTS ↔ Native（NAPI）**：当 Skill 需要 C/C++ 能力时，通过 NAPI 桥接。
- **意图框架 / Harmony Intelligence**：鸿蒙大模型智能体的"技能/意图"注册方式，研究 Skill 能否以"意图"形式注册（这是 Skill VM 落地的关键接口）。

> 以上每一条都可以作为一次文献/实验子任务，由 B3 与 A 组协同攻克。

## 5. 快速验证（可选软路由）

在尚未拿到鸿蒙真机/模拟器前，可在**本机 Docker/OpenHarmony 模拟镜像**里先验证纯逻辑层（IR、配置解析、协议），把鸿蒙特定部分（NAPI、ArkTS）隔离成适配器，在主开发机上持续集成。

## 6. 常见问题

| 问题 | 处理 |
| --- | --- |
| 模拟器启动卡住 | 检查是否勾选 HVD 组件、CPU 虚拟化（BIOS 开启 VT-x/AMD-V） |
| 签名报错 | 登录开发者联盟账号，自动签名即可 |
| ArkTS 语法报错 | ArkTS 是 TS 严格子集，禁用部分 `any`/动态属性，按官方规范修正 |

> 首次跑通 "Hello World" 的成员，把截图/现象记录到例会笔记，作为团队里程碑。

详细参考：华为开发者官网 https://developer.huawei.com/consumer/cn/ 与 OpenHarmony 官网 https://www.openharmony.cn/。
