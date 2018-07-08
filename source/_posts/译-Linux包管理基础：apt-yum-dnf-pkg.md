---
toc: true
title: '[译]Linux包管理基础：apt,yum,dnf,pkg'
date: 2018-05-05 15:51:14
tags: [linux,yum,apt]
---

> 原文地址: [Package Management Basics: apt, yum, dnf, pkg](https://www.digitalocean.com/community/tutorials/package-management-basics-apt-yum-dnf-pkg)
> 原文作者: Brennen Bearnes
> 译文出自: [Cap_ljf's Blog](http://cap-ljf.top/)
> 本文永久链接: 
> 译者: cap_ljf

### 引言
大多数类Unix操作系统提供了一个用于查找和安装软件的集中式机制。软件通常以软件包的形式分发，保存在软件仓库中。使用软件包被称为软件包管理。软件包为操作系统提供基本的组件，以及共享库、应用程序、服务和文档。
<!--more-->
软件包系统不仅仅是一次性安装软件。它还提供用于升级已安装软件的工具。软件包仓库有助于确保代码已经过审核以供你的系统使用，并且已安装的软件版本已获得开发人员和程序包维护人员的批准。

在配置线上或开发环境时，往往需要去官方仓库检查软件版本。已发行稳定版中的软件包可能已过时，特别是在涉及新的或快速变化的软件情况下。尽管如此，软件包管理对于系统管理员和开发者来说是一项至关重要的技能，而为主流发行版提供的丰富软件包是一项巨大的资源。

本指南旨在作为一个查找、安装和升级各个发行版软件包的基础知识的快速参考，并应帮助你在不同系统之间切换这些知识。

### 软件包管理系统：摘要
大多数软件包系统都是围绕软件包文件集建立的。一个软件包文件通常是一个文档，其中包含已编译二进制文件和其他组装软件的资源以及安装脚本。软件包还包含有价值的元数据，包括依赖项（安装和运行该软件依赖的其他包列表）。

尽管它们的功能和优点大致相同，但各种平台的封装格式和工具各不相同：

| Operating System      |    Format | Tool(s)  |
| :-------- | --------:| :--: |
| Debian  | .deb |  apt, apt-cache, apt-get, dpkg   |
| Ubuntu  | .deb |  apt, apt-cache, apt-get, dpkg   |
| CentOS  | .rpm |  yum   |
| Fedora  | .rpm |  dnf   |
| FreeBSD  | Ports,.deb |  make,pkg   |

在Debian和基于它的系统中，如Ubuntu、Linux Mint和Raspbian，包格式是`.deb`文件。高级打包工具APT（Advanced Packaging Tool）提供用于大多数常见操作的命令：搜索软件仓库、安装软件包集合以及它们的依赖项，以及管理升级。APT命令作为底层命令dpkg工具的前端操作，dpkg处理本地系统上各个`.deb`文件的安装，有时会直接调用dpkg。

近期大多数Debian派生的发行版系统包括`APT`命令，该命令为常用操作提供了一个简洁统一的接口，这些接口传统上由更具体的`apt-get`和`apt-cache`。它的使用是可选的，但可以简化一些任务。

CentOS、Fedora和其他Red Hat家族的成员使用RPM文件。在CentOS中，`yum`用于与单个软件包文件和仓库交互。

在最近的Fedora版本，`yum`已被`dnf`取代，`dnf`是`yum`一个现代化的分支，它保留了`yum`的大部分接口。

FreeBSD的二进制包系统使用pkg命令进行管理。FreeBSD还提供Ports Collection，一个本地目录结构和工具，允许用户使用Makefiles直接从源代码获取、编译和安装软件包。使用pkg通常要方便得多，但偶尔预编译的软件包不可用，或者你可能需要更改编译时选项。

### 更新软件包列表
大多数系统会保留一份远程仓库可用软件列表在本地数据库。你最好在安装和更新软件前更新这个数据库。作为这种模式的一个部分例外，`yum`和`dnf`会在执行某些操作之前检查更新，但你可以随时询问是否有更新可用。

| System      |    Command |
| :-------- | :--------|
| Debian/Ubuntu  | sudo apt-get update |
||sudo apt update|
| CentOS  | yum check-update |
|  Fedora | dnf check-update |
| FreeBSD Packages  | sudo pkg update |
| FreeBSD Ports  | sudo portsnap fetch update |

### 更新已安装软件

对于一个没有软件包系统的机器，确保所有已安装软件保持是最新版本将是一个巨大的负担。你将不得不追踪数百个不同软件包的上游更改和安全警报。虽然软件包管理器不能解决升级软件时遇到的每个问题，但它确实使你能够使用少量的命令来维护大多数系统组件。

在FreeBSD，升级已安装的ports可能导致重大的改变或需要手工配置步骤。在使用portmaster升级之前，最好先阅读/usr/ports/UPDATING。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu  | sudo apt-get upgrade |  只升级已安装的并且可以升级的软件包  |
|| sudo apt-get dist-upgrade | 可以添加或删除软件包以满足新的依赖关系 |
|| sudo apt upgrade | 同 sudo apt-get upgrade |
|| sudo apt full-upgrade | 同 apt-get dist-upgrade. |
| CentOS  | sudo yum update |  |
| Fedora | sudo dnf upgrade ||
| FreeBSD Packages	 | sudo pkg upgrade ||
| FreeBSD Ports | less /usr/ports/UPDATING | 使用`less`命令查看ports的更新注释 |
|| cd /usr/ports/ports-mgmt/portmaster && sudo make install && sudo portmaster -a | 安装portmaster并用它去更新已安装的ports |

### 查找一个软件包
大多数发行版都提供软件集的图形或菜单驱动前端。这些可以按类别浏览并发现新软件是一种好方法。但是，通常情况下，查找软件包的最快和最有效的方法是使用命令行工具进行搜索。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
|  Debian / Ubuntu | apt-cache search search_string |  |
|   | apt search search_string |  |
| CentOS  | yum search search_string |  |
|   | yum search all search_string | 搜索所有字段，包括软件名、描述等 |
| Fedora  | dnf search search_string |  |
|   | dnf search all search_string | 搜索所有字段，包括软件名、描述等 |
| FreeBSD Packages  | pkg search search_string | 根据软件名搜索 |
|   | pkg search -f search_string | 根据软件名搜索，返回完整的的描述 |
|   | pkg search -D search_string | 搜索描述 |
| FreeBSD Ports  | cd /usr/ports && make search name=package | 根据软件名搜索 |
|   | cd /usr/ports && make search key=search_string	 | 搜索评论、描述和依赖项 |

### 查看一个具体软件包的信息
当要决定安装哪一个软件包时，读一读软件包的详细描述会很有用。除了可读的文本之外，这些文件通常还包含元数据，如版本号和包的依赖项列表等。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu | apt-cache show package | 显示本地缓存该软件包的信息 |
|  | apt show package |  |
|  | dpkg -s package | 显示一个软件包现在的已安装状态 |
| CentOS | yum info package |  |
|  | yum deplist package | 软件包的依赖项列表 |
| Fedora | dnf info package |  |
|  | dnf repoquery --requires package | 软件包的依赖项列表 |
| FreeBSD Packages | pkg info package | Shows info for an installed package. |
| FreeBSD Ports | cd /usr/ports/category/port && cat pkg-descr |  |


### 从仓库安装一个软件包
一旦你知道软件包的名字，通常你可以使用单条命令来安装它和它的依赖项。一般来说，你可以将所有软件名一起列出将它们全部安装。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu | sudo apt-get install package |  |
|  | sudo apt-get install package1 package2 ... | 安装所有列出的软件 |
|  | sudo apt-get install -y package | 提示是否继续是都选择"yes" |
|  | sudo apt install package | 显示一个有颜色的进度条 |
| CentOS | sudo yum install package |  |
|  | sudo yum install package1 package2 ... | 安装所有列出的软件 |
|  | sudo yum install -y package | 提示是否继续是都选择"yes" |
| Fedora | sudo dnf install package |  |
|  | sudo dnf install package1 package2 ... | 安装所有列出的软件 |
|  | sudo dnf install -y package | 提示是否继续是都选择"yes" |
| FreeBSD Packages | sudo pkg install package |  |
|  | sudo pkg install package1 package2 ... | 安装所有列出的软件 |
| FreeBSD Ports | cd /usr/ports/category/port && sudo make install | 从源代码编译安装一个port |

### 从本地文件系统安装一个软件包

有时，即使软件没有针对给定的操作系统正式打包，开发者或供应商也会提供打包文件下载。你通常可以使用Web浏览器下载，或者通过`curl`命令下载。一旦安装包在目标系统上，它通常可以用一个命令安装。

在Debian派生的系统上，dkpg处理单个软件包文件。如果一个软件包为满足依赖关系，`gdebi`通常可以用来从官方存储库检索它们。

在CentOS和Fedora系统上，`yum`和`dnf`命令用于安装单个文件，这两个命令也会处理需要的依赖包。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu | sudo dpkg -i package.deb |  |
|  | sudo apt-get install -y gdebi && sudo gdebi package.deb | 安装`gdebi`并使用gdebi安装package.deb并安装它的依赖项 |
| CentOS | sudo yum install package.rpm |  |
| Fedora | sudo dnf install package.rpm |  |
| FreeBSD Packages | sudo pkg add package.txz |  |
|  | sudo pkg add -f package.txz | 即使软件已经安装了也依旧安装 |


### 卸载一个或多个已安装软件

由于软件包管理器知道给定软件包提供了哪些文件，因此如果软件不再需要，通常可以从系统中彻底删除它们。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu	 | sudo apt-get remove package |  |
|  | sudo apt remove package |  |
|  | sudo apt-get autoremove | 删除所有不再需要的包 |
| CentOS | sudo yum remove package |  |
| Fedora | sudo dnf erase package |  |
| FreeBSD Packages | sudo pkg delete package |  |
|  | sudo pkg autoremove | 删除所有不再需要的包 |
| FreeBSD Ports | sudo pkg delete package |  |
|  | cd /usr/ports/path_to_port && make deinstall | De-installs an installed port. |

### `apt`命令
Debian家族发行版的管理员通常熟悉`apt-get`和`apt-cache`。较少广为人知的是简化的apt接口，apt是专为交互式使用而设计的。

| Traditional Command      |    apt Equivalent |
| :-------- | :--------| 
| apt-get update | apt update |
| apt-get dist-upgrade | apt full-upgrade |
| apt-cache search string | apt search string |
| apt-get install package | apt install package |
| apt-get remove package | apt remove package |
| apt-get purge package | apt purge package |

尽管`apt`对于是一个更容易记住的操作，但是它并不是完全取代传统命令`apt-get`等的工具，并且它的接口可能会在不同版本之间进行更改以提高可用性。如果你在脚本或shell管道中使用包管理命令，那么坚持使用apt-get和apt-cache是一个好主意。

### 帮助
除了基于Web的文档之外，请记住，Unix手册（通常称为man pages）可用于shell中的大多数命令。要阅读一个页面，请使用`man`：
```bash
$ man page
```
使用`man`命令，你可以通过键盘箭头键进行导航，按`/`搜索页面中的文本，按`q`退出。

| System      |    Command | Notes  |
| :-------- | :--------| :--: |
| Debian / Ubuntu | man apt-get | 更新本地数据库包并使用软件包 |
|  | man apt-cache | 查询本地软件数据库 |
|  | man dpkg | 处理单个文件包并查询已安装的软件 |
|  | man apt | 查询一个对大多数操作更简洁的、用户友好的接口 |
| CentOS | man yum |  |
| Fedora | man dnf |  |
| FreeBSD Packages | man pkg | 处理已编译二进制包 |
| FreeBSD Ports | man ports | 处理Ports Collection. |

### 结论&扩展阅读
本指南提供了可以在系统间进行交叉引用的基本操作的概述，但只是浏览了一个复杂主题的表面。有关给定系统的更多细节，你可以参阅以下资源：
- [This guide](https://www.digitalocean.com/community/tutorials/ubuntu-and-debian-package-management-essentials)涵盖Ubuntu和Debian软件包管理器的细节。
- 这是一份[official CentOS guide to managing software with yum](https://www.centos.org/docs/5/html/yum/)
- There's a [Fedora wiki page about dnf](https://fedoraproject.org/wiki/Dnf), and [an official manual for dnf itself](https://dnf.readthedocs.org/en/latest/index.html).
- [This guide](https://www.digitalocean.com/community/tutorials/how-to-manage-packages-on-freebsd-10-1-with-pkg) covers FreeBSD package management using pkg.
- The [FreeBSD Handbook](https://www.freebsd.org/doc/handbook/) contains a [section on using the Ports Collection](https://www.freebsd.org/doc/handbook/ports-using.html).
