# OpenClaw 环境部署指南（Windows + WSL2 + Ubuntu）

>Author: 孙冉 241880144

本教程记录在 Windows 环境下通过 WSL2 部署 OpenClaw 的完整流程，适合团队成员快速复现开发环境。

主要包含：

- WSL2 与 Ubuntu 配置
- 将 Ubuntu 安装到自定义磁盘位置
- Linux 用户与 sudo 配置
- Node.js 环境安装
- OpenClaw 官方脚本安装
- Gateway 启动与验证
- Skills 检查
- LLM Provider 配置准备

> 推荐环境：
>
> - Windows 10 / Windows 11
> - WSL2
> - Ubuntu 22.04 LTS
> - Node.js ≥ 22
> - OpenClaw

---

## 1. 整体环境结构

推荐的部署结构如下：

```text
Windows
│
├── WSL2
│
└── Ubuntu 22.04
    │
    └── Linux 用户目录
        │
        ├── Node.js
        ├── npm
        ├── OpenClaw
        ├── Gateway
        ├── Skills
        └── LLM Provider 配置
```

如果不希望 Ubuntu 默认占用系统盘，可以通过 `wsl --import` 将整个 Ubuntu 文件系统安装到其他磁盘。

---

## 2. 检查 WSL 环境

在 PowerShell 中执行：

```powershell
wsl --status
```

查看当前 WSL 状态。

继续查看已经安装的发行版：

```powershell
wsl -l -v
```

如果显示：

```text
VERSION
2
```

说明当前使用的是 WSL2。

如果已经存在不需要保留的 Ubuntu 环境，可以先注销：

```powershell
wsl --unregister Ubuntu-22.04
```

如果还有其他 Ubuntu：

```powershell
wsl --unregister Ubuntu
```

> 注意：`wsl --unregister` 会删除该发行版中的所有 Linux 文件，请确认其中没有需要保留的数据。

---

## 3. 准备 Ubuntu 安装包

查看微软提供的 WSL 发行版：

```powershell
wsl --list --online
```

一般可以看到：

```text
Ubuntu
Ubuntu-22.04
Ubuntu-24.04
...
```

本教程采用：

```text
Ubuntu-22.04
```

### 3.1 下载 Ubuntu 22.04 WSL 安装包

创建一个临时下载目录，例如：

```powershell
mkdir D:\OpenClaw\Ubuntu
```

进入目录：

```powershell
cd D:\OpenClaw\Ubuntu
```

下载 Ubuntu 22.04：

```powershell
curl.exe -L -o Ubuntu2204.appx https://aka.ms/wslubuntu2204
```

下载完成后检查文件大小：

```powershell
dir
```

正常情况下应该是数百 MB 或 1 GB 左右，而不是几百字节。

---

## 4. 解压 Ubuntu Appx

PowerShell 的 `Expand-Archive` 默认只支持 `.zip`，因此先修改后缀：

```powershell
Rename-Item Ubuntu2204.appx Ubuntu2204.zip
```

创建解压目录：

```powershell
mkdir Ubuntu2204
```

解压：

```powershell
Expand-Archive -Path Ubuntu2204.zip -DestinationPath Ubuntu2204
```

进入目录：

```powershell
cd Ubuntu2204
```

通常可以看到类似文件：

```text
Ubuntu_2204.x.x.x_x64.appx
Ubuntu_2204.x.x.x_ARM64.appx
...
```

x64 Windows 应选择：

```text
Ubuntu_2204.x.x.x_x64.appx
```

---

## 5. 提取 Ubuntu RootFS

将 x64 Appx 改为 zip：

```powershell
Rename-Item Ubuntu_2204.x.x.x_x64.appx Ubuntu_2204_x64.zip
```

创建 RootFS 解压目录：

```powershell
mkdir ..\UbuntuRootfs
```

解压：

```powershell
Expand-Archive -Path Ubuntu_2204_x64.zip -DestinationPath ..\UbuntuRootfs
```

进入：

```powershell
cd ..\UbuntuRootfs
```

确认存在：

```text
install.tar.gz
```

这个文件就是 WSL 导入时需要的 Linux RootFS。

---

## 6. 将 Ubuntu 导入指定磁盘

创建 WSL 存储目录，例如：

```powershell
mkdir D:\OpenClaw\WSL\Ubuntu-22.04
```

执行导入：

```powershell
wsl --import Ubuntu-22.04 D:\OpenClaw\WSL\Ubuntu-22.04 D:\OpenClaw\Ubuntu\UbuntuRootfs\install.tar.gz --version 2
```

参数说明：

| 参数 | 含义 |
|---|---|
| `Ubuntu-22.04` | WSL 发行版名称 |
| `D:\OpenClaw\WSL\Ubuntu-22.04` | Ubuntu 实际存储位置 |
| `install.tar.gz` | Ubuntu RootFS |
| `--version 2` | 使用 WSL2 |

检查：

```powershell
wsl -l -v
```

应看到：

```text
NAME            STATE      VERSION
Ubuntu-22.04    Stopped    2
```

查看磁盘目录：

```powershell
dir D:\OpenClaw\WSL\Ubuntu-22.04
```

通常可以看到：

```text
ext4.vhdx
```

`ext4.vhdx` 就是整个 Ubuntu Linux 文件系统。

---

## 7. 第一次进入 Ubuntu

启动：

```powershell
wsl -d Ubuntu-22.04
```

第一次通过 `wsl --import` 导入后，一般会直接进入 root：

```bash
root@localhost:~#
```

检查：

```bash
whoami
```

输出：

```text
root
```

---

## 8. 创建普通 Linux 用户

不建议长期使用 root 用户运行 npm 或 OpenClaw。

创建用户：

```bash
adduser username
```

按照提示设置密码，其余信息可直接回车跳过。

加入 sudo 组：

```bash
usermod -aG sudo username
```

切换到普通用户：

```bash
su - username
```

检查：

```bash
whoami
```

应输出：

```text
username
```

检查家目录：

```bash
pwd
```

正常应为：

```text
/home/username
```

---

## 9. 设置默认登录用户

使用 root 编辑：

```bash
nano /etc/wsl.conf
```

加入：

```ini
[user]
default=username
```

保存并退出：

```text
Ctrl + O
Enter
Ctrl + X
```

退出 Ubuntu：

```bash
exit
```

回到 PowerShell：

```powershell
wsl --shutdown
```

重新进入：

```powershell
wsl -d Ubuntu-22.04
```

如果提示符变成：

```bash
username@localhost:~$
```

说明默认用户配置成功。

---

## 10. 检查 sudo 权限

执行：

```bash
sudo whoami
```

输入刚才设置的 Linux 用户密码。

如果输出：

```text
root
```

说明 sudo 配置正常。

---

## 11. 更新 Ubuntu

执行：

```bash
sudo apt update
```

然后：

```bash
sudo apt upgrade -y
```

在 WSL 中可能看到：

```text
invoke-rc.d: could not determine current runlevel
```

这通常与 WSL 的服务管理机制有关，只要升级最终正常完成即可。

---

## 12. 安装基础开发工具

安装：

```bash
sudo apt install -y curl git build-essential
```

验证：

```bash
curl --version
git --version
gcc --version
```

---

## 13. 安装 Node.js 22

项目要求 Node.js ≥ 22。

添加 NodeSource：

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
```

安装：

```bash
sudo apt install -y nodejs
```

检查：

```bash
node -v
npm -v
```

示例：

```text
v22.x.x
10.x.x
```

只要：

```text
Node.js >= 22
```

即可。

---

## 14. 使用官方脚本安装 OpenClaw

确保当前位于 Linux 家目录：

```bash
cd ~
pwd
```

应该显示：

```text
/home/username
```

运行 OpenClaw 官方安装脚本：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装结束后，脚本可能提示：

```text
PATH updated in ~/.bashrc
```

如果当前终端还无法识别 `openclaw`，执行：

```bash
source ~/.bashrc
```

然后检查：

```bash
openclaw --version
```

正常应显示类似：

```text
OpenClaw 2026.x.x
```

---

## 15. OpenClaw 初始化

安装完成后会自动进入初始化向导，也可以手动执行：

```bash
openclaw onboard
```

首次测试推荐：

```text
Quick start
```

Agent 类型选择：

```text
One agent
```

如果暂时不想配置云端模型：

```text
Skip for now
```

完成后通常会生成：

```text
~/.openclaw/
```

其中包含：

```text
~/.openclaw/openclaw.json
~/.openclaw/workspace
~/.openclaw/agents/
```

---

## 16. 启动 Gateway

由于 WSL 环境下可能没有可用的 service manager，因此 OpenClaw 可能提示：

```text
service installation skipped
```

这种情况下直接以前台方式运行 Gateway：

```bash
openclaw gateway run
```

正常启动后会看到类似：

```text
[gateway] starting HTTP server...
[gateway] http server listening
[gateway] ready
```

默认通常监听：

```text
127.0.0.1:18789
```

这个终端需要保持运行。

---

## 17. 检查 Gateway 状态

重新打开一个 Ubuntu 终端：

```powershell
wsl -d Ubuntu-22.04
```

执行：

```bash
openclaw gateway status
```

正常情况下应看到：

```text
Connectivity probe: ok
Listening: 127.0.0.1:18789
```

这说明：

```text
CLI
 ↓
Gateway
 ↓
通信正常
```

---

## 18. 检查 Skills

查看所有 Skill：

```bash
openclaw skills list
```

查看 Skill 状态：

```bash
openclaw skills check
```

输出可能类似：

```text
Total: 55
Eligible: 20
Visible to model: 20
Available as command: 19
Missing requirements: 35
```

其中：

```text
Ready and visible to model
```

中的 Skill 可以直接供 Agent 使用。

例如查看 weather：

```bash
openclaw skills info weather
```

可能显示：

```text
weather ✓ Ready
Visible to model: yes
Available as command: yes
```

---

## 19. Agent 调用方式

OpenClaw 的 Skill 通常不是通过：

```bash
openclaw skills run xxx
```

直接执行，而是：

```text
用户任务
  ↓
Agent
  ↓
模型判断
  ↓
自动选择 Skill
  ↓
执行 Skill
```

Agent 调用命令：

```bash
openclaw agent --message "你的任务"
```

例如：

```bash
openclaw agent --message "查询东京今天的天气"
```

如果尚未配置模型 Provider，可能出现：

```text
No route-compatible authentication source is configured
```

这说明：

```text
Gateway 正常
Agent 正常
Skill 正常
但还没有 LLM Provider
```

---

## 20. 配置 LLM Provider

后续可以分别配置多个 Provider，例如：

- Qwen / DashScope
- DeepSeek
- OpenAI
- OpenRouter
- Ollama

用于对比：

- 响应速度
- 中文能力
- 指令遵循
- Skill 调用稳定性
- 工具使用能力
- 成本

---

## 21. 配置 Qwen / DashScope API Key

在阿里云百炼中创建普通 API Key。

然后在 Linux 中配置环境变量：

```bash
nano ~/.bashrc
```

加入：

```bash
export DASHSCOPE_API_KEY=your_api_key
```

保存后：

```bash
source ~/.bashrc
```

检查：

```bash
openclaw models status
```

如果识别成功，可以看到类似：

```text
alibaba
qwen-dashscope
source=env: DASHSCOPE_API_KEY
```

> 注意：API Key 不要提交到 Git 仓库中。

---

## 22. 模型相关命令

查看模型状态：

```bash
openclaw models status
```

查看模型列表：

```bash
openclaw models list
```

刷新模型目录：

```bash
openclaw models refresh
```

设置默认模型：

```bash
openclaw models set <model-id>
```

查看帮助：

```bash
openclaw models --help
```

---

## 23. 常见问题

### 23.1 `Expand-Archive` 不支持 `.appx`

错误：

```text
.appx is not a supported archive file format
```

解决：

```powershell
Rename-Item xxx.appx xxx.zip
```

然后再：

```powershell
Expand-Archive
```

---

### 23.2 下载的 tar.gz 只有几百字节

如果：

```powershell
dir
```

发现下载文件只有：

```text
286 bytes
```

说明实际上下载到了：

```text
404 Not Found
```

HTML 页面，而不是 Ubuntu 镜像。

可以：

```powershell
notepad filename.tar.gz
```

检查文件内容。

如果看到：

```html
404 Not Found
```

说明下载地址已失效，应更换下载源。

---

### 23.3 `openclaw: command not found`

执行：

```bash
source ~/.bashrc
```

然后：

```bash
openclaw --version
```

如果安装脚本将 npm global bin 添加到了：

```text
~/.npm-global/bin
```

重新加载 `.bashrc` 后即可生效。

---

### 23.4 `wsl` 命令在 Ubuntu 中提示找不到

例如：

```text
Command 'wsl' not found
```

原因是：

```bash
wsl
```

是 Windows 命令，应在：

```text
PowerShell
```

中执行，而不是 Ubuntu shell 中。

---

### 23.5 Gateway 提示没有 Service Manager

例如：

```text
No supported service manager detected
```

在 WSL 中属于常见情况。

可以直接以前台方式运行：

```bash
openclaw gateway run
```

不影响 Gateway 正常使用。

---

### 23.6 Skill 显示 `needs setup`

表示 Skill 缺少额外依赖，例如：

- API Key
- 外部 CLI
- macOS 专属能力
- 第三方账号

查看具体要求：

```bash
openclaw skills info <skill-name>
```

或者：

```bash
openclaw skills check
```

---

## 24. 最终环境检查

建议依次执行：

```bash
node -v
npm -v
openclaw --version
openclaw gateway status
openclaw skills check
openclaw models status
```

如果前几项全部正常，说明 OpenClaw 基础环境已经配置完成。

推荐最终确认以下内容：

```text
[✓] WSL2 正常
[✓] Ubuntu 22.04 正常
[✓] Ubuntu 文件系统位于自定义磁盘
[✓] 普通 Linux 用户可用
[✓] sudo 正常
[✓] Node.js ≥ 22
[✓] npm 正常
[✓] OpenClaw CLI 正常
[✓] Gateway 可启动
[✓] Gateway connectivity probe 正常
[✓] Skills 可正常识别
[ ] LLM Provider 根据项目需求继续配置
```
