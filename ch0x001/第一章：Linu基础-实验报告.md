# Linux系统与网络管理实验

## 第一章：Linux基础（实验）

### 任务目标

- 调查并记录实验环境的如下信息：
  - 当前Linux发行版基本信息
  - 当前Linux内核版本信息
- Virtual box安装完Ubuntu后新添加的网卡如何实现系统开机自动启用和自动获取IP？
- 如何使用`scp`在「虚拟机和宿主机之间」、「本机和远程 Linux 系统之间」传输文件？
- 如何配置SSH免密登录？

---

### 完成情况

- [x] 记录当前Linux发行版基本信息
- [x] 当前Linux内核版本信息
- [x] Virtual box安装完Ubuntu后新添加的网卡实现开机自启和自动获取IP
- [x] 使用`scp`在「虚拟机和宿主机之间」、「本机和远程 Linux 系统之间」传输文件
- [x] 配置SSH免密登录

---

### 实验报告

#### 1. 调查并记录实验环境的如下信息

##### 1.1 查看当前Linux发行版基本信息

使用如下命令即可查看

```bash
lsb_release -a
```

查询结果如下

![](.\pictures\Find the version of Linux.png)

##### 1.2 查看当前Linux内核版本信息

使用如下命令即可查看

```bash
uname -a
```

查询结果如下

![KernalVersion](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\Find the kernel version of Linux.png)

---

#### 2. Virtual box安装完Ubuntu后新添加的网卡如何实现系统开机自动启用和自动获取IP？

假设当前虚拟机的网卡情况如下：

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\InternetCardInformation.png)

使用`ip a`命令查看当前系统内的网络情况

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\SystemInternetCardInformation.png)

现在，添加一块新网卡（NAT转换）

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\anothercard.png)

之后再次使用`ip a`指令查看

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\SystemInternetCardInformation.png)

可以看到，由于配置了DHCP服务器，并且采用冷启动方式，系统自动为新添加的网卡分配了一个IP地址和开机自启

---

#### 3. 使用`scp`在「虚拟机和宿主机之间」、「本机和远程 Linux 系统之间」传输文件？

首先，对于scp协议的介绍是这样的

> SCP（Secure Copy）是一种基于SSH协议的安全文件传输协议。它允许用户在本地主机和远程主机之间安全地复制文件和目录。
>
> SCP协议是SSH协议的一个子协议，由于SSH协议的普及，因此SCP协议也随之广泛应用。

对于`scp`的使用命令如下

```bash
scp myfile user@remote_host:/tmp/ # 本地文件传输到远程主机中
scp user@remote_host:/tmp/remote_file /local/dir/ # 远程主机传入本地
```

##### 3.1 虚拟机文件传输至宿主机

首先，在虚拟机`/home/test/`下创建文件`test.txt`并写入以下内容

```bash
test
```

然后，将其传入宿主机的桌面下

在传入宿主机桌面前，首先需要知道宿主机的IP地址

在Windows中，可以使用`ipconfig`查看

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\hostipaddress.png)

之后，可以使用`scp`进行文件传送了

在虚拟机中输入如下命令

```bash
scp test.txt yinya@192.168.100.138:C:/Users/yinya/Desktop/
```

遇到了问题

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\problem.png)

上网搜索解决方案，发现本机Windows中没有安装`OpenSSH-Server`服务

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\reason.png)

安装后继续尝试

后来发现还是不可以，依旧上网查找资料发现需要启动服务，并且打开22端口才可以

在Windows Terminal下输入 如下命令启动服务并开启22端口

```bash
# Start the sshd service
Start-Service sshd

# OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}

```

之后再次进行尝试

传输成功

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\success.png)

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\file.png)

##### 3.2 宿主机文件传输至虚拟机

在宿主机中新建文件`test2.txt`并输入以下内容

```bash
testtesttesttest
```

之后进行传输，输入以下命令

```bash
scp yinya@192.168.100.138:C:\Users\yinya\Desktop\test2.txt /home/test/ # 远程主机传入本地
```

成功

![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\transport2.png)

---

#### 4. 配置SSH免密登录

1. 宿主机创建一组公钥和私钥

   ```bash
   ssh-keygen
   ```

   ![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\kengen.png)

​		`ls`一下生成目录，可以看到出现了两个文件

​		<img src="C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\kengen2.png" style="zoom: 67%;" />

​		其中`id_rsa.pub`为公钥，需要上传至服务器（此处为虚拟机）中

2. 上传公钥

   在客户机中输入以下命令将公钥上传

   ```bash
   ssh-copy-id test@192.168.31.3
   ```

   遇到问题，提示命令不存在

   ![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\problem2.png)

   上网查找资料发现，需要使用`scp`指令上传

   ```bash
   scp C:\Users\yinya\.ssh\id_rsa.pub test@192.168.31.3:~
   ```

   将复制的公钥追加到`~/.ssh/authorized_keys`文件中

   ```bash
   cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

   修改权限

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

3. 查看结果

   退出原ssh连接，重新连接，无需输入密码，成功

   ![](C:\Users\yinya\Desktop\Linux系统与网络管理\pictures\success2.png)

---

### 参考资料

[Ubuntu开启SSH免密登录_ubuntu ssh免密登录_天雪浪子的博客-CSDN博客](https://blog.csdn.net/u010044182/article/details/128664248)

[Windows下安装openSSH_openssh windows_冬日小酒馆的博客-CSDN博客](https://blog.csdn.net/nl9788/article/details/131653284)

[远程 ssh 连接到 Windows 系统_ssh windows_慕伏白的博客-CSDN博客](https://blog.csdn.net/qq_43377653/article/details/130692431)

[SCP协议详细解析_笔记大全_设计学院 (python100.com)](https://www.python100.com/html/93278.html)

[ubuntu之间通过ip使用scp传输文件_ubuntu scp_不学无术杰哥的博客-CSDN博客](https://blog.csdn.net/balabala_333/article/details/130722169)

[server - ssh: connect to host myremotehost.com port 22: Connection refused - Ask Ubuntu](https://askubuntu.com/questions/144364/ssh-connect-to-host-myremotehost-com-port-22-connection-refused)

[ubuntu20添加新网卡后设置自动启用并获取ip_netplan开机自动激活网卡_xanarry的博客-CSDN博客](https://blog.csdn.net/xiongyangg/article/details/110206220)

[如何查看Linux系统版本和内核版本_linux教程_设计学院 (python100.com)](https://www.python100.com/html/62475.html)
