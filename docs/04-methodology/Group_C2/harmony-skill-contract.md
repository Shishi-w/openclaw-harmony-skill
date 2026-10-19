# 鸿蒙侧 Skill 接入契约文档 v1.0

> **所属**：C2 组 · 鸿蒙侧调研组
> **文档目标**：面向"OpenClaw Skill → HarmonyOS"移植研究，整理鸿蒙侧（HMAF / Ability Kit / 小艺开放平台）Skill 与智能体接入的完整契约。
> **覆盖范围**：`skillProfiles` 字段语义、`SKILL.md` 契约、ArkTS 入口脚本契约、权限模型、Agent / Skill / A2A 三种接入方式差异、与 OpenClaw `SKILL.md` 的字段映射。
> **版本**：v1.0（2026-10-17）
> **依据**：华为开发者官网文档（HarmonyOS API 26.0.0 起，API 版本以官网为准），参考链接见第 10 章。

---

## 1. 整体架构

### 1.1 调用链路总览

鸿蒙侧"应用 Skill"的完整调用链路如下（自 API 26.0.0 起，Ability Kit 支持将应用内业务能力以 Skill 形式开放给系统智能体）：

```
┌──────────────────────────────────────────────────────────────────────┐
│  用户自然语言输入                                                       │
└───────────────────────────────┬──────────────────────────────────────┘
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│  小艺（系统智能体）                                                     │
│  SDK 边界：小艺开放平台 / 系统级 Agent 能力                              │
│  ① 依据 SKILL.md 的 description + 触发场景做「意图—能力」初筛            │
│  ② 读取「执行参数」契约，把用户话术抽取为技能入参                         │
└───────────────────────────────┬──────────────────────────────────────┘
                                ▼  调用 Skill（能力单元，已注册到模块）
┌──────────────────────────────────────────────────────────────────────┐
│  Skill 注册层：module.json5 → skillProfiles[]                          │
│  SDK 边界：@ohos.app.ability.scriptManager（脚本管理）                  │
│  绑定 abilityName 指定的 UIAbility / ServiceExtension 上下文            │
└───────────────────────────────┬──────────────────────────────────────┘
                                ▼  加载 skills/<name>/scripts/*.ets
┌──────────────────────────────────────────────────────────────────────┐
│  ArkTS 入口脚本（薄适配层）                                             │
│  export default class + public async 方法（= SKILL.md functionName）    │
│  首参固定 ArkTSScriptInfo；基于 args Schema 校验入参                     │
└───────────────────────────────┬──────────────────────────────────────┘
                                ▼  import 应用内既有业务模块
┌──────────────────────────────────────────────────────────────────────┐
│  应用业务实现（例如 MusicPlayer.ets）              ← 与 Skill 机制解耦    │
│  结果经 ExecuteResult → completeArkTSScriptInApp 回传小艺               │
└──────────────────────────────────────────────────────────────────────┘
```

**面向移植研究的要点**：Skill 是"声明式能力外化"——描述文件（SKILL.md）声明触发场景、入参约束与返回值契约，ArkTS 入口脚本把外部调用桥接到应用内既有业务。开发者无需改造业务实现，系统智能体无需理解应用内部实现，仅依赖统一声明契约即可调度。

### 1.2 三种接入方式对比（Agent / Skill / A2A）

**这是 C2 组调研的关键产出**。鸿蒙智能体生态中有三种相互关联、但定位不同的接入方式：

| 维度 | ① 应用 Skill（基于 ArkTS 脚本） | ② Agent 接入（AgentExtensionAbility） | ③ A2A 接入（Agent 间协议 / HMAF） |
| --- | --- | --- | --- |
| **核心语义** | 把**应用内业务能力**声明式外化，供**系统智能体（小艺）**调用 | 把**应用作为智能体服务端**，供系统应用/其他 Agent 连接调用 | 智能体与智能体之间的**标准化互操作协议** |
| **最小版本** | API 26.0.0 起 | API 24 起（AgentExtensionAbility） | 端侧 A2A：API 24；AgentAbilityExtension 方案：API 26.0.0 起 |
| **注册/描述机制** | `module.json5 → skillProfiles[]` + `skills/<name>/SKILL.md` + ArkTS 入口脚本 | `module.json5 → extensionAbilities[]`（type=`agent`，绑定 `agent_config.json` → AgentCard） | AgentCard 能力发现 + A2A 协议（Task / Message / Artifact） |
| **调用方向** | 小艺（系统智能体）→ App 业务能力 | 客户端连接服务端 Agent，双向数据 + 可选双向认证 | 任意 Agent 客户端 ↔ Agent 服务端（A2A Server） |
| **开发者要写的** | SKILL.md + 一个薄 ArkTS 入口脚本 | AgentExtensionAbility 子类 + agent_config.json | A2A Server（`createA2AServer`）+ 任务管理逻辑 |
| **典型场景** | "放首歌""查天气"等系统级意图直达 App 能力 | 应用提供可被其他应用发现的 Agent 服务 | 旅行规划等跨 Agent 复合任务协作 |

> **关系澄清（易混淆点）**：
> - "Skill" 是**能力单元**（可被调用的业务封装），既依附于应用（本章讨论的 skillProfiles 方案），也作为 **AgentCard.skills[]** 出现在 Agent 的能力描述中。
> - "Agent" 是**主体**（能自主执行任务的组件）；"Skill" 是 Agent / 应用对外暴露的**能力**；"A2A" 是 Agent 之间**交互的协议**。
> - 小艺既可以作为系统智能体透过 `skillProfiles` 调度应用 Skill，也可以作为 A2A 客户端与其它 Agent 协作。

---

## 2. 目录结构契约

### 2.1 固定目录约定

官方要求（基于 ArkTS 脚本的应用 Skill 开发指导）：所有 Skill 相关文件必须放在**模块（Module）根目录**下的固定目录中：

```text
<Module>/
├── AppScope/
│   ├── app.json5
│   └── resources/
└── entry/                                  # 模块名（示例）
    ├── skills/                             # 【固定值】当前模块所有 Skill 的根目录
    │   └── <skill-name>/                   # Skill 目录名，必须与 SKILL.md 的 name 一致
    │       ├── scripts/                    # 【固定值】ETS 脚本目录
    │       │   └── <Skill>.ets             # Skill 入口脚本（srcEntries 指向此处）
    │       └── SKILL.md                    # 【固定值】Skill 描述文件
    └── src/
        └── main/
            ├── ets/
            │   ├── entryability/EntryAbility.ets
            │   └── service/<业务.ets>      # 应用内业务，被入口脚本 import
            ├── module.json5                # skillProfiles / requestPermissions 在此配置
            └── resources/
```

**关键约束（官方原文）**：`srcEntries` 指定的 `.ets` 文件必须放置在 `skills/{skill-name}/scripts/` 目录下，其中 `{skill-name}` 为 `skillProfiles` 中配置的技能名称；最多支持 100 个文件路径。

### 2.2 "三处 name 一致"规则

Skill 名称在 **三个位置必须完全一致**，不一致会导致注册或匹配失败：

| 位置 | 配置项 | 官方约束 |
| --- | --- | --- |
| ① `module.json5` | `skillProfiles[].name` | 仅小写字母/数字/连字符（`-`），以字母或数字开头/结尾，不能以连字符开头/结尾，不得出现连续连字符，≤64 字节，当前模块内唯一 |
| ② `SKILL.md` | frontmatter 的 `name` | 必须与 Skill 目录名以及 `skillProfiles[].name` 完全一致 |
| ③ 目录名 | `skills/<skill-name>/` | 建议使用公司/组织名作为前缀防止冲突（官方示例：`example-org-music-assistant`） |

> 命名建议：skill 名称采用**反向域名风格**（如 `example-org-music-assistant`），官方文档明确"为防止命名冲突，推荐使用公司或组织名作为前缀"。

### 2.3 示例（官方"音乐助手 Skill"）

```text
entry/
├── skills/
│   └── example-org-music-assistant/
│       ├── scripts/
│       │   └── MusicSkill.ets
│       └── SKILL.md
└── src/main/ets/service/MusicPlayer.ets
```

---

## 3. skillProfiles 字段语义表

`skillProfiles` 标签（API 26.0.0 起）标识当前模块的**技能配置信息**，用于定义 AI 代理（智能体）的技能能力。仅 `type` 取值为 `entry`、`feature`、`shared`、`skill` 的模块配置该标签生效。

### 3.1 字段语义表（官方 module.json5 配置 · 表32）

| 字段 | 类型 | 必填 | 语义 | 误用后果 | 官方文档出处 |
| --- | --- | --- | --- | --- | --- |
| `name` | string | 是 | 技能名称，当前模块内唯一。仅允许小写字母、数字、连字符 `-`；必须以小写字母或数字开头/结尾；不得以连字符开头/结尾、不得连续连字符；最大 64 字节。 | 命名不合法 → 构建或注册失败；与 `SKILL.md`、目录名不一致 → 意图匹配不上 | module.json5 配置文件 → skillProfiles 标签 |
| `abilityName` | string | 否 | 与该技能关联的组件名称，必须配置为 `abilities` 下的 UIAbility 或 `extensionAbilities` 下 `type` 为 `service` 的 ServiceExtension 组件名。≤127 字节，以字母开头，可含字母/数字/下划线/点。**仅 `entry`/`feature`/`shared` 模块适用；`skill` 类型模块不支持**。 | 指向不存在/类型不符的组件 → 技能无法挂载到可运行上下文 | 同上 |
| `srcEntries` | string[] | 否 | 实现技能的代码文件路径列表（`.ets`），每个元素相对当前模块的 `skills` 目录；文件须位于 `skills/{skill-name}/scripts/`；最多 100 个。 | 路径不符 → 脚本找不到，入口类不加载 | 同上 |
| `permissions` | string[] | 否 | 调用该技能所需权限列表。**当其他应用调用该技能时**，需要申请相应权限；元素≤255 字节，取值参考应用权限列表。 | 权限未声明 → 其他应用调用被拒绝 | 同上 |
| `version` | string | 是 | 技能版本号，格式 `主.次.补丁`（如 `1.0.1`），各版本号非负整数且不能以 0 开头（本身为 0 除外）。 | 格式错误 → 构建 / 分发校验失败 | 同上 |
| `visibility` | string | 否 | 技能可见性：`private`（仅当前应用）/ `system`（系统应用+当前应用）/ `public`（所有应用）。**缺省值 `system`**。 | 可见性设置错误 → 其他应用发现不了，或过度暴露能力 | 同上 |

### 3.2 官方完整示例

```json5
{
  "module": {
    // ...
    "skillProfiles": [
      {
        "name": "example-org-music-assistant",
        "abilityName": "EntryAbility",
        "version": "1.0.0",
        "visibility": "public",
        "srcEntries": [
          "../../skills/example-org-music-assistant/scripts/MusicSkill.ets"
        ]
      }
    ],
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

## 4. SKILL.md 契约

`SKILL.md` 是 Skill 的**声明契约文件**，是系统智能体进行"意图—能力"语义匹配的**唯一依据**。正文由"元数据 → 触发场景 → 能力契约"三段构成。

### 4.1 Frontmatter 必填/可选字段

| 字段 | 必填 | 语义与约束 |
| --- | --- | --- |
| `name` | 是 | 必须与 Skill 目录名、`skillProfiles[].name` 完全一致（见 2.2）。 |
| `description` | 是 | 简洁描述能力范围，是系统智能体**初次筛选**的关键依据，直接影响召回率。 |

```text
---
name: example-org-music-assistant
description: 提供音乐搜索播放与播控能力，响应"放首歌"、"切歌"、"暂停"等播放控制类指令
---
```

### 4.2 正文"能力契约"写法

正文分三段：

1. **触发场景（## 触发场景）**：用自然语言列出典型话术（覆盖多种说法），并显式列出**不要调用的情况**以划清能力边界、降低误触发。

   ```text
   ## 触发场景

   当用户明确表达**播放音乐**或**控制当前播放**时调用。典型话术：

   - "放一首SingerA的《SongA》"
   - "暂停"、"继续播放"、"切歌"、"下一首"、"上一曲"

   不调用的情况：

   - "把这首歌加入收藏" —— 意图是修改歌单，本 Skill 仅支持播放与播控。
   - "调小音量" —— 意图是系统音量控制，应走系统能力。
   ```

2. **执行参数（### 场景N：能力名（functionName）的子小节）**：每个能力一个子小节，含 `exec-cli` 形式调用示例 + 一份 JSON Schema（核心是 `args`）。

   调用示例四要素：`command` 固定为 `ohos-arkTSScript`；`skillName` 与 `SKILL.md` name 一致；`scriptPath` 为相对 Skill 目录的入口脚本路径；`functionName` 与 `.ets` 中 public 方法名**严格对应**。

   ```bash
   exec-cli(command: ohos-arkTSScript --skillName 'example-org-music-assistant' --scriptPath 'scripts/MusicSkill.ets' --functionName 'playMusicByName' --args '{
       "arg1": "SongA",
       "arg2": "SingerA"
   }')
   ```

   ```json
   "args": {
     "type": "object",
     "properties": {
       "arg1": { "type": "string", "description": "歌曲名，如《SongA》" },
       "arg2": { "type": "string", "description": "歌手名，如SingerA" }
     },
     "anyOf": [
       { "required": ["arg1"] },
       { "required": ["arg2"] }
     ]
   }
   ```

   > 注意 `argN` 的**位置语义**：入口脚本按 `argv` 位置取值，`arg1` 对应 `argv[0]`，依此类推。

3. **执行返回值**：列出**所有可能的结果分支**（成功 + 各类失败），再给整体 JSON Schema（顶层定义 `type`/`status` 等公共字段，用 `oneOf` 枚举分支形态）。

   官方示例的分支集合（`playMusicByName`）：

   | 分支 | status | 标志字段 |
   | --- | --- | --- |
   | 成功播放 | `success` | `data.playingTrack`、`data.matchedCount` |
   | 入参非法 | `failed` | `errCode: ERR_INVALID_PARAMS`、`errMsg`、`suggestion` |
   | 未命中 | `failed` | `errCode: ERR_NOT_FOUND`、`data.searchedKeywords`、`suggestion` |
   | 内部错误 | `failed` | `errCode: ERR_INTERNAL`、`errMsg`、`suggestion` |

   ```json5
   {
     "type": "object",
     "required": ["type", "status"],
     "properties": {
       "type":   { "type": "string", "const": "result" },
       "status": { "type": "string", "enum": ["success", "failed"] },
       "data":   { "type": "object" },
       "errCode": {
         "type": "string",
         "enum": ["ERR_INVALID_PARAMS", "ERR_NOT_FOUND", "ERR_INTERNAL"]
       },
       "errMsg":     { "type": "string", "minLength": 1 },
       "suggestion": { "type": "string", "minLength": 1 }
     },
     "oneOf": [
       /* 成功：要求 data.playingTrack 与 data.matchedCount */
       /* ERR_INVALID_PARAMS：要求 errMsg 与 suggestion */
       /* ERR_NOT_FOUND：要求 data.searchedKeywords 与 suggestion */
       /* ERR_INTERNAL：要求 errMsg 与 suggestion */
     ]
   }
   ```

### 4.3 description / 触发场景与语义匹配的关系

| 环节 | 使用的 SKILL.md 信息 | 作用 |
| --- | --- | --- |
| **初筛（召回）** | frontmatter `description` | 系统智能体据此判断"该 Skill 是否与当前意图相关"；描述越准确，召回率越高 |
| **精筛（区分/边界）** | 触发场景的典型话术 + "不调用的情况" | 划清能力边界，降低相邻意图的误触发 |
| **入参抽取** | 执行参数 `args` JSON Schema（含 `description`） | 指导智能体把用户话术映射为结构化入参（`arg1`/`arg2`...） |
| **结果消费** | 执行返回值 Schema + 示例 | 智能体把扣回的结构化结果转成自然语言回复 |

---

## 5. ArkTS 入口脚本契约

`SKILL.md` 声明"契约"，`ArkTS 入口脚本`给出"实现"，二者互为表里。

### 5.1 代码骨架契约

- **导出方式**：`export default class` 导出一个类。
- **方法映射**：类内每个 `public async` 方法对应 `SKILL.md` 中声明的一项能力（`functionName`），方法名必须与 `SKILL.md` 中的 `functionName` **严格一致**。
- **签名约定**：第一个参数类型固定为 `scriptManager.ArkTSScriptInfo`，用于接收系统传入的脚本上下文；其后可追加自定义参数（通过 `...argv: string[]`）。

```typescript
import { scriptManager } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';
// 应用既有业务模块（Skill 与业务解耦的关键：直接 import）
import { MusicPlayer, Track, PlayResult } from '../../../src/main/ets/service/MusicPlayer';

export default class MusicSkill {
  public async playMusicByName(info: scriptManager.ArkTSScriptInfo, ...argv: string[]): Promise<void> {
    // 1) 按位置解析 argv，对照 SKILL.md 的 args Schema 做前置校验
    // 2) 调用应用内业务实现（如 MusicPlayer.searchAndPlay）
    // 3) 把业务返回值/异常映射为 SKILL.md 声明的分支，构造 ExecuteResult 回传
  }
  public async controlPlayback(info: scriptManager.ArkTSScriptInfo, ...argv: string[]): Promise<void> {
    // 同上模式
  }
}
```

### 5.2 与业务模块的 import 关系

- 入口脚本位于 `skills/<name>/scripts/`，通过相对路径 `import` 应用内既有业务模块（如 `../../../src/main/ets/service/MusicPlayer`）。
- 入口脚本**不承载业务逻辑**，只做"参数适配器"：解析并校验入参 → 调用业务 → 把业务返回值与运行时异常分别映射到 `SKILL.md` 声明的不同结果分支。

### 5.3 结果回传契约（scriptManager）

| API / 结构 | 说明 |
| --- | --- |
| `scriptManager.ArkTSScriptInfo` | 入口函数第一参数；字段：`requestCode: string`（请求码，只读）、`context: Context`（绑定的 Ability 上下文，只读）。系统能力 `SystemCapability.Ability.AgentRuntime.Core`，仅 Stage 模型。 |
| `scriptManager.ExecuteResult` | 执行结果结构：`code: number`（结果码，默认 0）、`result: Record<string, Object>`（可选）、`uris: string[]`（可选，授权给调用方的 URI）、`flags: number`（URI 读写权限，与 Want flags 一致）。 |
| `completeArkTSScriptInApp(context, requestCode, result): Promise<void>` | 唯一的回包出口：完成脚本执行并上报结果。常见错误码：`16000020`（非 Ability 上下文）、`16000003`（ID 不存在）、`16000050`（内部错误）。 |

```typescript
private async report(info: scriptManager.ArkTSScriptInfo, result: scriptManager.ExecuteResult): Promise<void> {
  try {
    await scriptManager.completeArkTSScriptInApp(info.context, info.requestCode, result);
  } catch (e) {
    const err = e as BusinessError;
    console.error(`completeArkTSScriptInApp failed, code: ${err.code}, message: ${err.message}`);
  }
}
```

### 5.4 入参校验与分支映射要点（来自官方示例）

- 空参/非法参 → 走 `ERR_INVALID_PARAMS` 分支，**不再进入业务**。
- 枚举参数（如播控动作）→ 用白名单二次校验。
- 业务异常（`catch`）→ 统一映射 `ERR_INTERNAL` 分支。
- 回包内容（`status` / `errCode` / `data` / `suggestion`）须与 `SKILL.md` "执行返回值"声明的分支**保持一致**。

---

## 6. 权限模型

### 6.1 requestPermissions 与 skillProfiles.permissions 的区别

| 对比项 | `module.json5 → requestPermissions` | `skillProfiles[].permissions` |
| --- | --- | --- |
| **语义** | 标识**当前应用运行时**需向系统申请的权限集合（如 `ohos.permission.INTERNET`）。 | 标识**调用该技能**所需要的权限；**当其他应用调用该技能时**，需要申请相应权限。 |
| **作用方向** | 本应用 → 系统（申请能力） | 他应用/系统智能体 → 本技能（调用门槛） |
| **配置位置** | `module` 顶层 | `module.skillProfiles[]` 内 |
| **示例** | `{ "name": "ohos.permission.INTERNET" }`（Skill 访问云端列表需要） | `"permissions": []` 或具体权限名（元素≤255 字节，参考应用权限列表） |

> 简单记：`requestPermissions` 是"我要用系统的什么"，`skillProfiles.permissions` 是"谁来调我的技能得先有什么"。二者都**声明**在 `module.json5`，但 `requestPermissions` 通常还会配合运行时提权（`@ohos.abilityAccessCtrl`/UI 授权）使用。

### 6.2 visibility 对可见/可调范围的控制

| 取值 | 可见范围 | 典型用途 |
| --- | --- | --- |
| `private` | 仅当前应用可见 | 应用内部自用能力 |
| `system`（**缺省**） | 系统应用 + 当前应用可见 | 开放给系统智能体（小艺）调用的常规配置 |
| `public` | 所有应用都可见 | 需要被第三方应用发现/调用的能力 |

### 6.3 预置应用与签名要求（官方约束）

- 官方 module 配置明确：`skillProfiles` **仅对 `type` 为 `entry`/`feature`/`shared`/`skill` 的模块生效**。
- 当应用的 `bundleType` 配置为 `skill`（`app.json5`）时，才允许 `Module` 的 `type` 配置为 `skill`，且**此时应用只能包含 1 个 Module**，`abilityName` 字段对 `skill` 类型模块**不支持**。
- 该标签在部分场景**仅对预置应用生效**——是否所有免预置应用都能上架开放 Skill，是需要与官方确认的关键点（见第 9 章）。

### 6.4 待确认：AGENT_SKILL 权限

模板初稿提及 "`AGENT_SKILL` 权限"。经查官方 `skillProfiles` 标签与 `requestPermissions` 文档，**未发现名为 `AGENT_SKILL` 的标准权限**；调用侧/Agent 场景的权限以 A2A 协议的双向安全认证（`onAuth` / `authorize`）与 `skillProfiles.permissions` 覆盖。是否仍存在系统级隐藏权限，列入疑问清单待真机验证。

---

## 7. 契约变更实验记录

> 模板约定的实验编号（E1–E7）。**当前阶段（v1.0）基于官方 API 26.0.0 文档完成静态契约梳理**，真机/样例实验尚未在本仓库跑通，以下先给"已确认事实"与"建议实验设计"，实验完成后回填结论。

### 7.1 已确认的版本演进事实（API 版本对照）

| 能力 | 起始版本 | 说明 |
| --- | --- | --- |
| `AgentExtensionAbility` 组件（智能体服务端） | API 24 | 端侧智能体框架（端侧 A2A）能力延伸 |
| `agent_config.json` / AgentCard | API 24（AgentCard 细分为 APP/ATOMIC_SERVICE 在 API 26 起） | 智能体能力描述 |
| 基于 ArkTS 脚本的应用 Skill（`skillProfiles`） | API 26.0.0 | 本次契约核心 |
| HMAF（Agent Framework Kit）A2A 协议、AgentAbilityExtension、Function 组件 | API 26.0.0 | 智能体间通信、拉起 |

### 7.2 建议实验设计（E1–E7，占位待填）

| 编号 | 实验内容 | 验证点 | 状态 |
| --- | --- | --- | --- |
| E1 | 空 Skill Demo 注册 | `skillProfiles` 三处 name + `srcEntries` 路径能否被识别 | 待做（对应 A2 组 Demo） |
| E2 | description 召回 | 改变 SKILL.md description，验证小艺意图匹配召回差异 | 待做 |
| E3 | 入参校验 | 构造非法 `argN`，验证 `ERR_INVALID_PARAMS` 分支回传 | 待做 |
| E4 | 返回值分支 | 成功 / 未命中 / 内部错误三分支回传与自然语言化 | 待做 |
| E5 | 权限控制 | 设置 `skillProfiles.permissions` + `visibility`，从第三方/系统侧验证调用门槛 | 待做 |
| E6 | A2A 服务端 | `createA2AServer` 跑通 Execute/Cancel/PerceptionSuggest | 待做 |
| E7 | 跨设备/预置 | 非预置应用是否可上架开放 Skill（预置约束） | 待做 |

---

## 8. 与 OpenClaw SKILL.md 的字段映射（初稿）

OpenClaw / Agent Skills 规范（`agentskills.io`）的 `SKILL.md` = YAML frontmatter + Markdown 正文；鸿蒙侧"基于 ArkTS 脚本的应用 Skill"有着**惊人相似的声明式设计**，映射如下（初稿，待 E1–E4 实验校正）：

| OpenClaw / AgentSkills 字段 | 鸿蒙对应物 | 语义差异 | 移植注意事项 |
| --- | --- | --- | --- |
| frontmatter `name` | `skillProfiles[].name`、`SKILL.md` frontmatter `name`、Skill 目录名 | 鸿蒙要求小写字母/数字/连字符命名 + 三处一致 | 移植时统一改写三处；反向域名前缀可保留 |
| frontmatter `description` | `SKILL.md` frontmatter `description` | 二者都用于"初筛/召回"，但鸿蒙还有"触发场景"精筛层 | 不宜照搬长描述，应针对鸿蒙意图匹配改写 |
| 正文 instructions（步骤说明） | `SKILL.md` 触发场景 + 能力契约 | OpenClaw 正文是"教智能体怎么做"；鸿蒙是"声明能做什么 + 怎么调"（函数级契约） | **核心差异**：鸿蒙不含 step-by-step 指令，而是 JSON Schema 契约，需将 OpenClaw 技能改写为函数化能力 |
| 依赖门控 `requires.bins/env/config` | `module.json5 → requestPermissions` + 业务模块 import | OpenClaw 是运行时门控；鸿蒙是静态声明权限 | 鸿蒙侧没有 shell 级依赖，外部二进制需改为端侧服务/云能力 |
| `os` 平台门控（darwin/linux/win32） | 设备类型 `deviceTypes` / `appInfo.deviceTypes` | 鸿蒙面向 phone/tablet/2in1/tv/wearable 等 | 平台判定逻辑在鸿蒙无对应物 |
| `scripts/` 可执行脚本 | `skills/<name>/scripts/*.ets` 入口脚本 | OpenClaw 脚本自主执行；鸿蒙脚本是被系统智能体回调的薄适配层 | 入口函数签名必须遵循 `ArkTSScriptInfo` + 返回值契约 |
| 工具调用（宿主工具） | 应用内部业务能力（import 业务模块） | OpenClaw 有宿主工具集；鸿蒙由应用自身业务承担 | 需把工具调用改写为应用内业务 API |
| 加载优先级（workspace>project>personal>bundled） | `visibility`（private/system/public）+ `skillProfiles` 注册 | 鸿蒙按可见性 + 系统智能体匹配，无目录优先级 | 移植时把优先级语义转换为可见性/检索范围 |
| （无直接对应） | `version`、`abilityName`、`srcEntries` | 鸿蒙特有：版本、绑定组件、脚本路径 | 新增字段必须补全 |

> **移植结论（C2 视角）**：OpenClaw Skill 的"元数据 + 指令正文"结构与鸿蒙 `SKILL.md`（元数据 + 触发场景 + 函数契约）形态接近，**字段/目录骨架可映射**；但**执行模型差异大**——OpenClaw 是"指令驱动的自主多步执行"，鸿蒙是"契约驱动的单函数回调"。方向 A 移植时，每个 Skill 需做"函数化拆分"（一个能力 → 一个 `public async` 方法 + 一个 Schema 段），并补全 `version` / `visibility` / `permissions`。

---

## 9. 疑问清单

（周五分享会抛出；需真机验证或咨询官方后回填）

1. **预置应用约束范围**：`skillProfiles` 是否只对预置应用生效？第三方普通应用上架后能否被系统智能体（小艺）调用？能否通过 AGC 平台申请？
2. **`visibility` 缺省 `system` 的实际行为**：非系统应用将自己的 Skill 设为 `system` 是否有效？`public` 是否需要额外权限/审核？
3. **`skillProfiles.permissions` 的运行时语义**：`permissions` 是"调用方需持有的权限"还是"挂载的 Ability 需申请的权限"？系统智能体调用时如何授这个权？（官方文档表述偏"调用方需申请"，待实验确认）
4. **`AGENT_SKILL` / Agent 相关系统权限**：是否存在未公开系统级权限？A2A `onAuth`/`authorize` 双向认证的实现细节与 SKILL 权限的关系？
5. **`SKILL.md` 的能力数量上限**：一个 Skill 目录可声明多少个 `functionName`（场景子小节）？入口类多方法是否都支持任意数量 `...argv`？
6. **ArkTS 脚本的宿主能力边界**：入口脚本能否调用 `@kit.*` 全部能力（网络/文件/数据库）？还是受 `AgentRuntime` 沙箱限制？`completeArkTSScriptInApp` 临时文件授权（`uris`/`flags`）的实际用法。
7. **A2A 与 App Skill 的调度关系**：系统智能体在什么情况下走 `skillProfiles` Skill、什么情况走 A2A 连 `AgentExtensionAbility`？二者是否会同时触发、优先级如何？
8. **`ohos-arkTSScript` 的 `exec-cli` 参数**：`command` 的取值集合、`skillName` 是否区分大小写、`scriptPath` 是否允许跨模块引用？
9. **与 OpenClaw 的移植验证**：真实 OpenClaw Skill（如 file-reader）改写为鸿蒙 Skill 后，schema 契约能否被系统智能体正确消费（语义匹配效果需实测）。

---

## 10. 参考链接

> 访问日期：2026-10-17（HarmonyOS API 26.0.0 / HarmonyOS 6 系列文档，以官网"开发者文档中心"实时版本为准）。

| # | 文档 | URL |
| --- | --- | --- |
| 1 | 基于 ArkTS 脚本的应用 Skill 开发指导 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-skill-development-guide |
| 2 | module.json5 配置文件（skillProfiles 标签 / 表32） | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/module-configuration-file |
| 3 | @ohos.app.ability.scriptManager（脚本管理）API 参考 | https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-scriptmanager |
| 4 | 端侧 A2A 框架概述 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/agent-overview |
| 5 | 使用 AgentExtensionAbility 组件实现智能体服务 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/agent-extension-ability |
| 6 | AgentExtensionAbility 配置文件说明（agent_config.json / AgentCard） | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/agent-extension-configuration |
| 7 | Agent Framework Kit 简介 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hmaf-introduction |
| 8 | 通过 AgentAbilityExtension 实现智能体间 A2A 协议通信 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hmaf-a2a-dev-guide |
| 9 | 通过 Function 组件拉起智能体 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hmaf-function |
| 10 | Agent Framework Kit 术语 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hmaf-glossary |
| 11 | 小艺开放平台（云 A2A / 端 A2A 模式 Agent） | https://developer.huawei.com/consumer/cn/doc/doccenter-celia/cloud-a2a-0000001193466742 |
| 12 | Agent Skills（OpenClaw / AgentSkills.io 规范） | https://agentskills.io |
| 13 | 导入 Skill（服务端操作，待真机验证版块） | https://developer.huawei.com/consumer/cn/doc/service/import-skill-0000002592931542 |
| 14 | Skill 真机测试 | https://developer.huawei.com/consumer/cn/doc/service/skill-real-machine-testing-0000002592771628 |

---

*本文档由 C2 组（鸿蒙侧调研组）维护，版本 v1.0；未尽事项见第 9 章疑问清单，待真机实验后更新至 v1.1。*
