# A2 《DevEco 环境手册》

> 本文记录 A2 鸿蒙环境组的环境搭建和最小 Demo 验证情况。此版本只提交这一份文档，不附截图或新增 Demo 文件；请以步骤和文字结果记录验证过程。
>
> 当前进度：成员反馈已完成 DevEco 安装配置；HelloWorld 工程为 Stage / API 26。已收到 HAP 构建成功日志和 MatePad Pro 13 模拟器显示 Hello World 的截图。空 Skill 尚未通过系统智能体实际调用；A2 组内其他成员的进展仍待确认。

## 1. 目标与范围

记录如何配置 DevEco Studio（HarmonyOS API 26 / Stage 模型），如何在 MatePad Pro 13 模拟器上运行 Hello World，以及后续如何记录 `skillProfiles` 空 Skill Demo 的验证结果。

- 目标 API：API 26（对应 SDK 的具体版本和项目配置见下文）
- 项目模型：Stage（已由 HelloWorld 工程的 `apiType: "stageMode"` 确认）
- 模拟器：MatePad Pro 13
- Skill Demo：本机 HelloWorld 工程已有 `empty-skill-demo` 文件和注册配置；工程不在本仓库内，运行状态待本人验证

## 2. 当前进度

| 项目 | 状态 | 说明 |
| --- | --- | --- |
| DevEco Studio 安装与基本配置 | 已完成（成员反馈） | 已安装并完成基本配置 |
| HarmonyOS SDK / API 26 | 已安装（成员反馈） | HarmonyOS SDK 26.0.0 Release；工程 target / compatible SDK 均为 26.0.0 |
| Stage 项目配置 | 已确认 | `entry/build-profile.json5` 中 `apiType` 为 `stageMode` |
| MatePad Pro 13 模拟器 | 已使用 | 镜像为 HarmonyOS 7.0.0 (26.0.0) |
| Hello World HAP 构建 | 已成功 | 默认与 tablet 目标均显示 `BUILD SUCCESSFUL`，进程退出码 0 |
| Hello World 模拟器运行 | 已成功 | 已收到 MatePad Pro 13 模拟器显示 Hello World 的截图 |
| Skill Demo 文件与注册配置 | 已存在，配置可构建 | `SKILL.md`、ArkTS 入口和 `skillProfiles` 已配置；系统智能体调用待验证 |
| Skill Demo 系统调用 | 待验证 | 当前没有小艺 / 系统智能体返回结果 |
| A2 组内 Skill Demo 状态 | 待确认 | 暂无信息判断是否有其他成员跑通 |

## 3. 环境信息

| 项目 | 实际信息 |
| --- | --- |
| 操作系统 | Windows 11，OS Build `26200.9711` |
| DevEco Studio | `26.0.0 Release`，内部版本号：`26.0.0.851` |
| HarmonyOS SDK | `HarmonyOS SDK 26.0.0 Release`，`API Version 26.0.0 Release` |
| 项目模型 | Stage（`entry/build-profile.json5` 中 `apiType: "stageMode"`） |
| compile SDK | HarmonyOS SDK 26.0.0 Release（DevEco 当前配置；工程文件未显式写出 `compileSdkVersion`） |
| target SDK | `26.0.0`（`build-profile.json5`） |
| compatible SDK | `26.0.0`（`build-profile.json5`） |
| 模拟器设备 | MatePad Pro 13 |
| 模拟器镜像版本 / API | `HarmonyOS 7.0.0 (26.0.0)` |
| 验证日期 | `2026-10-15` |

## 4. 安装与基础配置

### 4.1 获取和安装 DevEco Studio

1. 从[华为开发者官网下载 DevEco Studio](https://developer.huawei.com/consumer/cn/deveco-studio/)。
2. 安装后首次启动，记录完整版本号和 Build 号：`26.0.0 Release`。
3. 在 SDK Manager 中确认并安装本次使用的 HarmonyOS SDK / API 26：`HarmonyOS 7.0.0 (26.0.0)`。
4. 本机曾将 DevEco 的 `bin` 目录加入 PATH；具体目录和用途待核对。若其他成员不需要该配置，应注明为本机可选设置，不要作为必需安装步骤。

### 4.2 配置 MatePad Pro 13 模拟器

1. 在 DevEco Studio 打开 `Tools > Device Manager`。
2. 选择或创建 MatePad Pro 13 模拟器，并下载与本次项目目标 API 相符的镜像。
3. 记录模拟器镜像显示的 HarmonyOS 版本和 API：`HarmonyOS 7.0.0 (26.0.0)`。

> 模拟器创建步骤和设备模板以当前 DevEco Studio 界面为准；参见[华为官方创建模拟器指南](https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/ide-emulator-create)。

### 4.3 安装结果确认

- [x] DevEco Studio 可以正常启动（成员反馈）。
- [x] HarmonyOS SDK / API 26 已安装（成员反馈）。
- [x] MatePad Pro 13 模拟器已用于运行 Hello World（成员反馈）。
- [x] Hello World 项目已确认为 Stage，target / compatible SDK 均为 API 26。

## 5. Hello World 验证

### 5.1 创建或打开项目

- 项目模板名称：`新建项目中的 Empty Ability`
- 项目模型：Stage；`entry/build-profile.json5` 中为 `"apiType": "stageMode"`
- compile SDK：HarmonyOS SDK 26.0.0 Release（由 DevEco 当前配置提供）
- target SDK：`26.0.0`；compatible SDK：`26.0.0`（根目录 `build-profile.json5`）
- 入口 Ability：`EntryAbility`；`entry/src/main/module.json5` 中配置，源码类继承 `UIAbility`
- 项目名称：`Hello World`

Stage 模型不需要单独安装。核对本工程时，以 `entry/build-profile.json5` 中的 `apiType: "stageMode"` 为直接依据；`module.json5` 中的 `type: "entry"` 表示入口模块类型，不表示 FA 模型。入口代码 `EntryAbility.ets` 继承 `UIAbility`，并在 `onWindowStageCreate` 中加载页面，与该工程配置相符。

### 5.2 运行步骤

1. 在 DevEco Studio 中打开 Hello World 项目。
2. 在设备选择栏选择 MatePad Pro 13 模拟器。
3. 点击运行，等待模拟器启动并完成应用部署。
4. 如果模拟器提示“快照启动失败，尝试冷启动”，按提示等待冷启动完成；设备就绪后再次点击运行。
5. 确认模拟器界面显示 `Hello World`。

### 5.3 结果记录

- 预期结果：应用成功启动，模拟器界面显示 `Hello World`。
- 实际结果：MatePad Pro 13 模拟器显示 `Hello World`；已收到运行截图。
- HAP 构建：默认目标和 tablet 目标均成功，进程退出码均为 `0`。
- 备注：截图通过对话提供，不存入仓库。

## 6. `skillProfiles` 空 Skill Demo

> 本地工程已经构建成功并在模拟器中运行 Hello World；这只证明应用 HAP 可构建和启动，不代表系统智能体已成功调用空 Skill。A2 组内其他成员是否跑通仍待确认。

### 6.1 Demo 来源与结构

- Demo 来源：本机 `HelloWorld` Stage 工程；当前工程目录不在 `openclaw-harmony-skill` 仓库内
- Skill 名称：`empty-skill-demo`
- Skill 目录：`entry/skills/empty-skill-demo/`
- 入口文件 / 方法：`scripts/EmptySkill.ets` / `runDemo`
- 注册配置：`entry/src/main/module.json5`
- 运行依赖、设备或权限：无网络和业务权限依赖；使用 MatePad Pro 13 模拟器进行本地应用构建 / 启动确认

### 6.2 注册配置

- `skillProfiles` 配置位置：`entry/src/main/module.json5` 的 `module` 对象
- 配置依据：[华为 ArkTS Skill 开发指南](https://developer.huawei.com/consumer/cn/doc/doccenter-capabilities/arkts-skill-development-guide)
- 当前字段与实际值：

| 字段 | 当前值 | 校对点 |
| --- | --- | --- |
| `name` | `empty-skill-demo` | 与 Skill 目录名、`SKILL.md` frontmatter 一致 |
| `abilityName` | `EntryAbility` | 与 `module.json5` 中的 Ability 名称一致 |
| `srcEntries` | `../../skills/empty-skill-demo/scripts/EmptySkill.ets` | 从 `entry/src/main/module.json5` 指向入口脚本 |
| `version` | `1.0.0` | Skill 版本 |

本示例没有 `permissions` Skill Profile 字段。若 Skill 以后需要系统权限，应在 `module.json5` 的模块级 `requestPermissions` 中单独声明；当前固定回包不需要额外权限。

当前注册片段：

```json5
"skillProfiles": [
  {
    "name": "empty-skill-demo",
    "abilityName": "EntryAbility",
    "srcEntries": [
      "../../skills/empty-skill-demo/scripts/EmptySkill.ets"
    ],
    "version": "1.0.0"
  }
]
```

`SKILL.md` 声明无参数的 `runDemo` 调用，入口脚本通过 `completeArkTSScriptInApp` 回传成功结果：`status: success`，`data.message: "empty-skill-demo ok"`。调用契约已补 JSON Schema；运行结果仍待验证。

### 6.3 已完成的本地构建和运行

本机在 DevEco 工具链中分别构建默认目标和 tablet 目标：

```text
默认目标：--mode module -p product=default -p module=entry@default assembleHap --analyze=normal --parallel --incremental --daemon
tablet 目标：--mode module -p module=entry@default -p product=default -p requiredDeviceType=tablet assembleHap --analyze=normal --parallel --incremental --daemon
```

两次构建均显示 `BUILD SUCCESSFUL`，进程退出码为 `0`。默认目标耗时约 237 ms（33 项任务，16 executed / 17 up-to-date）；tablet 目标耗时约 1.974 s（33 项任务，17 executed / 16 up-to-date）。

两次日志都包含以下 warning：`Will skip sign 'hos_hap'. No signingConfigs profile is configured in current project.` 这是签名配置缺失提示；此次构建仍成功，后续上传 HAP 或真机调试若遇到签名要求，再单独配置签名。

运行截图显示 MatePad Pro 13 模拟器已启动 Hello World。华为文档说明，DevEco 的 Run 操作会触发应用编译构建和推包，完成后应用运行在模拟器上。[官方模拟器运行说明](https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/ide-emulator-create)

### 6.4 什么算跑通

| 阶段 | 成功判据 | 能证明什么 / 不能证明什么 |
| --- | --- | --- |
| 阶段 | 成功判据 | 当前结果 | 能证明什么 / 不能证明什么 |
| --- | --- | --- | --- |
| 构建成功 | Build 窗口显示 `BUILD SUCCESSFUL`（或本地化等效提示），进程退出码为 0，并生成 HAP。Warnings 单独记录，不等同于构建失败。 | **已通过**：默认与 tablet 目标均成功；有签名配置 warning。 | 证明工程代码和配置完成编译打包；不能证明 Skill 已被系统智能体调用。 |
| 模拟器运行成功 | Run 完成安装 / 启动，MatePad Pro 13 模拟器显示 Hello World。 | **已通过**：用户提供的截图显示应用运行。 | 证明 HAP 可以安装运行；不能单独证明 Skill 可调用。 |
| Skill 配置一致 | HAP 构建成功，且 profile 名称、Ability、入口脚本与 `SKILL.md`、ArkTS 方法名一致。 | **配置已核对；本次 HAP 构建成功**。 | 证明工程声明和路径可构建；不是 Skill 调用成功的证据。 |
| Skill 系统调用 | 通过小艺开放平台导入 HAP，并按官方流程在支持的真机上测试；调用 `empty-skill-demo` 后看到 `empty-skill-demo ok`。 | **待验证**。 | 只有此项能证明系统智能体端到端调用及回包成功；模拟器运行不替代此项。 |

官方 Skill 流程包括导入 HAP 和真机测试。[华为 ArkTS Skill 开发指南](https://developer.huawei.com/consumer/cn/doc/doccenter-capabilities/arkts-skill-development-guide)

### 6.5 运行后请回传的信息

已收到构建日志和模拟器截图，截图不存入仓库。本次仓库提交仍只包含手册这一份文档。后续如继续验证 Skill 系统调用，请回传：

1. 小艺开放平台导入 HAP 的结果；若被签名配置拦截，提供完整提示。
2. 真机型号和 HarmonyOS 版本。
3. 调用 `empty-skill-demo` 后是否返回 `empty-skill-demo ok`；成功或失败结果的截图 / 日志。

## 7. 常见问题（FAQ）

| 现象 | 当前记录 / 处理方式 | 状态 |
| --- | --- | --- |
| 模拟器提示“快照启动失败，尝试冷启动” | 本机观察：等待模拟器完成冷启动，设备就绪后再次点击运行可显示 Hello World。若仍失败，记录完整报错和实际处理步骤。 | 本机经验，尚未交叉复现 |
| `Will skip sign 'hos_hap'` | 工程未配置 `signingConfigs`，日志为跳过签名 warning；本次 HAP 构建仍显示成功且模拟器已运行。若后续 HAP 导入或真机调试提示签名问题，再补配签名并记录结果。 | 原因已由构建日志确认；上传 / 真机签名要求待验证 |
| 其他问题 | `[遇到后补充现象、环境、原因及处理方式；尚无时写“暂无已记录问题”]` | `[待填写]` |

## 8. 验收清单

- [x] DevEco Studio 安装配置完成（成员反馈）。
- [x] 已安装 HarmonyOS SDK / API 26（成员反馈）。
- [x] 已使用 MatePad Pro 13 模拟器跑通 Hello World（成员反馈）。
- [x] 已核对并记录 DevEco 完整 Build 号、模拟器镜像版本、项目 Stage / compile / target API。
- [x] 默认与 tablet 目标 HAP 均构建成功；退出码均为 0。
- [x] HAP 已在 MatePad Pro 13 模拟器运行并显示 Hello World；截图已通过对话提供。
- [x] 已记录签名 warning：当前项目没有 `signingConfigs`，构建跳过 `hos_hap` 签名。
- [ ] 小艺开放平台导入及真机 Skill 调用已完成；如当前无真机，明确标为待验证。
- [ ] 已确认 A2 组内是否有其他成员跑通，并记录可复现的 Demo 来源。
- [ ] 已记录 Skill 注册判据和实际结果；如支持调用，也记录调用结果。
- [ ] 手册步骤已由另一位组员复现。
- [ ] FAQ 已记录实际遇到的问题；没有问题时注明“暂无已记录问题”。

## 9. 参考资料

| 标题 | 链接 | 用途 |
| --- | --- | --- |
| DevEco Studio 下载 | [华为开发者官网](https://developer.huawei.com/consumer/cn/deveco-studio/) | 获取开发工具 |
| DevEco Studio 26.0.0 发布说明 | [华为官方发布说明](https://developer.huawei.com/consumer/en/doc/harmonyos-releases/deveco-studio-new-features-2600) | 核对工具、SDK 与 API 版本 |
| HarmonyOS 7.0 升级适配指南 | [华为官方指南](https://developer.huawei.com/consumer/en/doc/harmonyos-releases/upgrade-adaptation) | 核对 HarmonyOS 7.0 与 API 26 的对应关系 |
| 创建 HarmonyOS 模拟器 | [华为官方指南](https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/ide-emulator-create) | 配置设备模板和镜像 |
| ArkTS 应用 Skill 开发指南 | [华为官方指南](https://developer.huawei.com/consumer/cn/doc/doccenter-capabilities/arkts-skill-development-guide) | 核对 `SKILL.md`、`skillProfiles`、ArkTS 入口及导入 / 真机验证流程 |

## 10. 修订记录

| 日期 | 修改人 | 修改内容 |
| --- | --- | --- |
| `2026-10-15` | `李朝阳` | 记录 DevEco 环境与 Hello World 进度；补充 MatePad Pro 13 模拟器信息，Skill Demo 保持待验证 |
| `2026-10-15` | `李朝阳` | 根据 HelloWorld 工程核实 Stage / API 26；补全空 Skill 配置和构建成功判据，执行结果待本人回传 |
