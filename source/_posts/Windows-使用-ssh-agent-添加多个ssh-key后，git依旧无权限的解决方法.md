---
title: Windows 使用 ssh-agent 添加多个ssh key后，git依旧无权限的解决方法
date: 2021-10-25 13:47:54
tags:
---
## 问题表现
在windows安装完ssh-agent，添加多个ssh key之后,git clone依旧提示Permission Denied。
（安装参见：[https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)）

## 问题原因
Git需要知道Windows ssh-agent服务的位置， 可以通过添加```GIT_SSH```环境变量来解决该问题。

以下文字来源：[https://snowdrift.tech/cli/ssh/git/tutorials/2019/01/31/using-ssh-agent-git-windows.html](https://snowdrift.tech/cli/ssh/git/tutorials/2019/01/31/using-ssh-agent-git-windows.html)

>Git for Windows uses the ssh binaries included with git by default. While this works well enough in most situations, one side-effect is that git has no idea how to talk to the Windows ssh-agent service. In order for git commands to use the Windows ssh-agent service, git needs to be informed of the system OpenSSH path. To accomplish this, the environment variable ```GIT_SSH```needs to be set with the path of the system OpenSSH executable.


## 解决方法
>Run the following command to update the environment variable:
>在PowerShell中运行以下命令添加环境变量：

```powershell
PS >  [Environment]::SetEnvironmentVariable("GIT_SSH", "$((Get-Command ssh).Source)", [System.EnvironmentVariableTarget]::User)
```