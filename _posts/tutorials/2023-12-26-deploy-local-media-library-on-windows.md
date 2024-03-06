---
title: 在Windows物理机上布置媒体库 - 使用WSL2, Jellyfin
categories: [Blogging, Media Library]
tags: [tutorial, media library]

toc: true

---

> 我期望在本地局域网内搭建一套媒体库，旨在随时随地访问我的电影和电视剧内容。这套方案将分为两个版本：
> 1. **验证版**：该版本将在实验室的电脑上持续运行，用于稳定性验证等。
> 2. **待定板**：该版本将在未来出现在我家里，现在还是一个TODO的状态。
{: .prompt-info}

<br />
## 验证版

<br />

### 环境和软件
- 红米笔记本（CPU Intel-Core i5-9300H@2.40GHz，RAM 16G）
- Jellyfin（[Jellyfin Server](https://github.com/jellyfin/jellyfin)、[Jellyfin Client AndroidTV](https://github.com/jellyfin/jellyfin-androidtv)、[Jekkyfin Client Android](https://github.com/jellyfin/jellyfin-android)）
<br />


### 预备内容

一些前期准备内容，包括搭建有效的局域网通信，以处理我的Shadowsocks无法在当作服务器的笔记本上运行的奇怪问题。
<br />


#### WSL2开启ssh server
<br />


##### 参考
- [wsl---ssh远程连接及ip映射配置详解](https://zhuanlan.zhihu.com/p/372601715)
- [使用 WSL 访问网络应用程序](https://learn.microsoft.com/zh-cn/windows/wsl/networking?source=recommendations)
<br />


##### 配置ssh服务器
由于该笔记本是首次用作服务器，因此我直接重装代替更新：
1. 卸载`openssh-server`：
    ```shell
    $ sudo apt remove openssh-server
    ```
2. 安装`openssh-server`：
    ```shell
    $ sudo apt install openssh-server
    ```
3. 配置文件：
    ```shell
    $ vim /etc/ssh/sshd_config
    ```
4. 配置IP和Root，允许用户名和密码登录，开放或者修改如下内容：
    ```shell
    Port 2222 # 配置ssh端口
    ListenAddress 0.0.0.0

    PermitRootLogin yes
    PasswordAuthentication yes
    PermitEmptyPasswords no
    ```
5. 重启ssh服务：
    ```shell
    sudo service ssh --full-restart
    ```
<br />


##### “固定”WSL IP

- [wsl2hosts](https://github.com/iamqiz/bash-wsl2-host/blob/main/README-CN.md)

根据这篇教程，在WSL2`.bashrc`里配置启动项，启动WSL的同时写Windows物理机hosts文件，因此需要一些辅助功能，比如类似于`sudo`的`gsudo`，直接照着这个仓库抄就行了。
<br />


##### 配置局域网访问

1. 配置Windows物理机到WSL端口转发规则：（TODO：目前Windows机重启后转发会失效，感觉可以在WSL的bashrc里配置）
    ```shell
    $ netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=[IP] connectport=[PORT]
    ```
2. 配置防火墙出入站规则：
    ```shell
    $ netsh advfirewall firewall add rule name=WSL2 dir=in action=allow protocol=TCP localport=2222
    ```
<br />


#### 远程传递文件

如果不能远程传递文件，那么服务器就没有意义，不如搭一个本地的。我们选用`rsync`进行文件的传递，速度取决于网口带宽：
```shell
$  sudo rsync -av -e "ssh -p 2222" ./Shadowsocks-4.4.1.0.zip  root@192.168.1.112:/mnt/c/Downloads/
```
考虑到我更改了ssh的服务端口，所以需要`-e "ssh -p 2222"`来指定rsync的端口。
<br />


### Jellyfin安装
<br />


## 附录：一些小工具
<br />


### 批量改名

使用[PowerToys](https://learn.microsoft.com/zh-cn/windows/powertoys/)和PowerShell配合批量替换文件名或者添加公共前缀，PowerShell的一些功能[subexpression operator](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators?view=powershell-7.2#subexpression-operator--)：
```shell
$(Get-ChildItem ./) | Rename-Item -NewName {"prefix" + $_.Name}
```
> `Get-ChildItem`会循环读取路径下的子文件/文件夹，因此需要增加`()`对方法进行限定。
{: .prompt-warning}

