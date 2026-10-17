# 对于孙冉同学的OpenClaw部署指南的补充（一些可选的操作）

> 作者：邱莉扉 241250097

这份补充主要有以下两个内容，属于可选操作，是个人经验摸索的总结：
1. 在wsl中配置自动启动openclaw的gateway
2. 在Windows主机中安装桌面端的openclaw app,并连接wsl中的gateway

## 1. 配置在进入wsl中自动启动openclaw的gateway
**为了让每次进入wsl就让gateway自动启动，免得每次还要手动去打开，可以在wsl中敲以下几行命令进行配置**

```wsl
systemctl --user status

```
输出中应该包含`Active: active (running)` 或 `State: running`的信息

如未安装或很久没更新gateway版本，请安装/更新gateway服务：
```wsl
openclaw gateway install
```
接着启用linger：
```wsl
sudo loginctl enable-linger "$(whoami)"
```
可能会让你输入密码，输入即可，你输入的密码应该不会显示在命令行里
验证是否生效：
```wsl
loginctl show-user "$USER" | grep Linger
```
预期看到`Linger=yes`，这样就生效了
接着敲入这一行，是让gateway在wsl登录时自动启动的关键命令：
```wsl
systemctl --user enable --now openclaw-gateway
```
最后验证配置是否成功：

确认服务已启用：
```wsl
systemctl --user is-enabled openclaw-gateway
```
期望输出：`enabled`

确认服务正在运行:
```wsl
systemctl --user status openclaw-gateway --no-pager
```
期望输出中包含绿色的`Active: active (running)`

确认 Gateway 可达
```wsl
openclaw gateway probe
```
期望输出中包含：`Gateway Status：Reachable: yes`，就ok了
- 注：每次在进入wsl后gateway自动启动需要大概1分钟左右的时间，需要等一小会再敲`openclaw gateway probe`来验证gateway是否连通

---

## 2.在Windows中安装桌面端app，并连接之前在wsl中安装好的gateway

**如果不习惯在漆黑的命令行里面使用openclaw，这是一份Windows桌面端app安装、gateway连接指南**

安装包发布页：https://github.com/openclaw/openclaw-windows-node/releases/tag/v2026.9.8-1
滑到最底下，根据你的电脑操作系统(arm/x86)选择下载:
`OpenClawCompanion-Setup-arm64.exe` 或 `OpenClawCompanion-Setup-x64.exe` 安装包

---

安装完之后打开OpenClaw,它会有一些set up的选项：
- 第一页选择默认的那一项即可；
- 在第二页的gateway那里，选择第二个 `Connecting to an existing Gateway`，点击右下角NEXT，会进入一个让你填写现有gateway信息的界面
- gateway地址填写：`ws://127.0.0.1:18789`
- 设置代码空着就行
- Token从wsl中获取：敲入` cat ~/.openclaw/openclaw.json | grep -A2 '"auth"' `,在输出中找到：
```wsl
"auth": {
      "mode": "token",
      "token": "*********************************************"
```
复制这串token填过去就行
- SSH不用管，空着就行
- 点最下面的`检查连接`，第一次连不通是正常的，因为在wsl中还没有完成配对。
  到wsl里去敲下
```wsl
openclaw devices list
```
输出一般为空
```wsl
openclaw devices approve --latest
```
会输出这一句：
`Selected pending device request <一串request-id>`，复制这串request-id,
命令行敲入（注意不用敲那对尖括号 <> ）：
```wsl
openclaw devices approve <上面复制过来的request-id>
```
再次验证：
```wsl
openclaw devices list
```
这时表格里会出现你刚刚那个设备 ，就完成了配对
- 重新在openClaw 的setup界面点击`检查连接`，这时就可以连接成功了

- 后面几页的配置比较简单按需填写即可

- 之后在使用桌面端app之前需要提前在命令行里进入wsl让gateway连接

这份文档纯属个人摸索折腾的经验，如有实操起来不一致的地方请查阅官方文档或咨询ai，感谢你能看到这里！