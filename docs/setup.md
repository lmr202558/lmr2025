# 本机开发环境记录

记录这台 Windows 机器上为开发工作做的配置，方便换机或重装时复现。

## 1. Git

| 项 | 值 |
| --- | --- |
| 版本 | Git for Windows 2.56.0.4 |
| 安装位置 | `C:\Program Files\Git` |
| 系统 PATH | `C:\Program Files\Git\cmd` |
| 用户 PATH | `C:\Program Files\Git\ucrt64\bin`（GCM 所在） |
| 提交身份 | `lmr202558 <lmr202558@users.noreply.github.com>` |

安装方式：

```powershell
winget install --id Git.Git --source winget
```

## 2. 凭据管理

使用 Git Credential Manager（GCM 2.9.1），通过 GitHub OAuth 设备码授权，
令牌由 Windows 凭据管理器托管，磁盘上不存明文。

```ini
# ~/.gitconfig
[credential]
	helper = manager
```

检查当前凭据：

```bash
printf "protocol=https\nhost=github.com\n\n" | git credential fill
```

## 3. GitHub CLI

| 项 | 值 |
| --- | --- |
| 版本 | gh 2.103.3 |
| 位置 | `C:\Program Files\GitHub CLI\gh.exe` |
| 登录账号 | `lmr202558` |
| Token scopes | `gist`, `read:org`, `repo` |

```powershell
winget install --id GitHub.Cli --source winget
```

## 4. SSH

GitHub 的 22 端口在本机被代理阻断（`Connection closed ... port 22`），
因此 `~/.ssh/config` 里把 `github.com` 指向 `ssh.github.com:443`：

```sshconfig
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
```

密钥类型 `ed25519`，之后所有 `git@github.com:owner/repo.git` 形式的地址都能直接用，
不需要改成 `ssh://...:443` 那种写法。

验证：

```bash
ssh -T git@github.com
# Hi lmr202558! You've successfully authenticated, but GitHub does not provide shell access.
```

## 5. dsh-ops 插件

DeepSeek Harness 的 token 效率插件 `dsh-ops`。

| 项 | 值 |
| --- | --- |
| 版本 | **0.4.4** |
| 安装位置 | `~\.dsh\profiles\desktop` |
| 宿主版本 | DSH 0.2.1-rc.5 |

> **版本注意**：dsh-ops 0.5.x 要求 `dsh >=0.2.2-rc.1`，0.4.x 支持 `0.2.0-rc.2 ~ 0.2.1`。
> 在本机 DSH 升级到 0.2.2 之前，必须锁在 0.4.4。

安装：

```powershell
& "D:\Users\lby\AppData\Local\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd" `
  plugin --profile desktop add dsh-ops@0.4.4
```

卸载：

```powershell
& "D:\Users\lby\AppData\Local\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd" `
  plugin --profile desktop remove dsh-ops
```

## 6. 代理

本机走 Clash 系 TUN + 系统代理 `127.0.0.1:7897`，DNS 返回 `198.18.x.x` 的 fake-ip。
命令行工具（git / curl / gh）默认继承系统代理即可联网。

---

<sub>最后更新：2026-11-12</sub>
