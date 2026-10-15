# 环境搭建指南：OpenClaw

> 本指南协助每位成员在本机安装 OpenClaw，并理解其 Skill 机制（研究对象的"源码级"第一课）。

## 1. OpenClaw 是什么

OpenClaw 是开源自主 AI 智能体框架（原 Clawdbot，作者 Peter Steinberger），运行在本机或云端，通过"Skill"让大模型能调用工具、完成任务。

- 官网：https://openclaw.ai
- 文档：https://docs.openclaw.ai
- GitHub：https://github.com/openclaw/openclaw

## 2. 前置要求

- Node.js ≥ 20（推荐用 Node 22 LTS）
- 一个可用的模型 API（OpenAI 兼容接口 / 其他 provider），或本地模型（llama.cpp/Ollama 等）
- 网络通畅（安装依赖需要）

检查：

```bash
node --version
npm --version
```

## 3. 安装 OpenClaw（推荐 npm 全局安装）

```bash
npm install -g @openclaw/openclaw    # 具体包名以官方文档为准
openclaw --version
```

> **重要**：以官方文档 https://docs.openclaw.ai/zh-CN/install 为准。若官方改为其他安装方式（Homebrew / 二进制），按官方最新说明操作。

## 4. 初始化与配置

```bash
openclaw init
openclaw doctor        # 环境自检：查看缺了什么依赖/配置
```

配置文件位于 `~/.openclaw/openclaw.json`（或工作区 `.openclaw/`），用于配置：

- 模型 provider 与 API Key
- `skills.entries.<skill>.enabled` 开关 Skill
- 沙箱、网关等高级选项

> 该项目为**公开仓库**，任何包含 API Key 的配置文件**一律 gitignore**（仓库已配置 `.gitignore`），切勿提交。

## 5. 动手：体验 Skill 机制（必做小任务）

1. 创建一个测试工作区目录：
   ```bash
   mkdir ~/openclaw-ws && cd ~/openclaw-ws
   ```
2. 在工作区 `skills/` 下新建一个最小的 Skill：
   ```
   skills/hello/SKILL.md
   ```
   内容：
   ```markdown
   ---
   name: hello
   description: 打招呼演示 Skill，用于验证 OpenClaw Skill 加载机制。
   ---

   当用户说"你好"或要求打招呼时，回复一句简短的问候，并说明这是 hello Skill 在起作用。
   ```
3. 运行 OpenClaw 并提问，观察该 Skill 是否被加载进提示词、是否生效。
4. 记录你观察到的现象（如门控/加载优先级的影响），在例会上分享。

> 给方向 B 的组员任务：再用 `openclaw skills list`、`openclaw skills check` 等命令查看 Skill 列表与依赖检查机制，尝试修改 `os` 字段观察加载结果，理解"平台门控"是如何工作的。

## 6. 卸载 / 重装（可选）

```bash
npm uninstall -g @openclaw/openclaw
```

## 7. 环境答疑

- 安装或启动报错，先在本仓库提 [Issue](https://github.com)（用 Issues 记录，便于归档），或在群里 @C1 环境组。
- `openclaw doctor` 的输出贴在 Issue 里，方便排查。
