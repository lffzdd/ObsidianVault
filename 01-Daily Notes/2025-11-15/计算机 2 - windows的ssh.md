Windows 当然也有 SSH 服务器，而且它的出现背后有一段“小小的技术演化史”。让我们从底层动机讲起，这样你能真正理解它为什么存在、为什么现在是这个样子。

---

## 1. **为什么 Windows 需要 SSH Server？（动机）**

SSH 本质上是“安全的远程登录与命令执行渠道”。  
Unix/Linux 世界早就把远程管理当成日常：管理员靠 `ssh` 登录服务器、跑脚本、传文件。

但在 Windows 早期（XP、Win7 那个时代），微软的远程管理哲学完全不同：

- Windows 偏向 GUI 远程桌面（RDP）
    
- 批处理与 PowerShell 虽然强，但远程执行能力有限
    
- 企业环境更多依赖 WinRM（Windows Remote Management）
    

问题是：  
**当 Windows 进入云时代（尤其加入 Azure 后），服务器需求变了——你必须让 Windows Server 也能像 Linux 那样被 SSH 管理。**

云端混合环境里，如果 Linux 和 Windows 都能用统一工具（SSH）管理，会极大降低运维成本。

于是，微软终于走向一个新方向：

**把 OpenSSH 原生集成进 Windows。**

---

## 2. **关键历史节点（演化故事）**

你能看到一个原本“不属于 Windows 世界”的工具，如何正式被 Windows 采用。

### ● **2015–2017：微软决定支持 OpenSSH**

微软宣布将 OpenSSH 移植到 Windows，并且由 PowerShell 团队维护。  
这是一件象征意义极强的事——微软开始全面拥抱开源。

### ● **2018–2019：Windows 10 加入可选功能 “OpenSSH Server”**

Windows 10 和 Windows Server 2019 都出现了可勾选安装的：

- OpenSSH Client
    
- OpenSSH Server
    

这意味着：  
**SSH Server 成为 Windows 的一等公民，不再需要第三方工具。**

### ● **现在：Windows 默认自带 SSH Server 组件**

在 Win10 1809+、Win11、Server 2019+ 中：

你可以直接用内置服务：

```
sshd
```

启动 Windows 的 SSH 服务器。

---

## 3. **它是怎么工作的？（核心思想）**

微软的 SSH Server 实际上就是 **OpenSSH 的 Windows 移植版本**，只是做了这些适配：

- 使用 Windows 的权限系统（NTFS ACL）来管理授权
    
- 用 Windows 服务的方式运行 `sshd`
    
- 默认账户认证基于系统账号（如本地用户或域用户）
    
- 公钥文件放在用户目录下的：
    

```
C:\Users\<User>\.ssh\authorized_keys
```

真正的工作方式跟 Linux 上的 OpenSSH 没什么本质差别，思想依旧是：

1. 客户端发起连接
    
2. 服务器端 `sshd` 处理加密握手
    
3. 验证密码或公钥
    
4. 进入一个 Shell 环境（在 Windows 上是 PowerShell）
    

---

## 4. **你实际能用的：Windows 自带 SSH Server**

查看是否安装：

```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```

安装：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

启动服务：

```
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

然后就可以从其他电脑 SSH 登录 Windows：

```
ssh username@windows-ip
```

这就是 Windows 上的 SSH 服务器程序。

---

## 5. **总结：为什么 Windows 有 SSH 服务器？**

根本原因是 **时代变了**。

- 企业需要统一的跨平台运维方式
    
- Windows 必须和云环境 Linux 系统协同
    
- 微软也拥抱了开源生态
    
- SSH 是事实上的远程管理标准
    

于是一个来自 Unix 世界的工具被“驯化”到 Windows 世界，最终成了官方内置的系统组件。

---

如果你愿意，我们还可以继续讲：  
为什么 Windows 远程管理最初选择 WinRM 而不是 SSH？  
OpenSSH 的 Windows 版本里究竟改了哪些底层实现？  
还是你想看看如何在 Windows 上做 SSH 公钥认证？