---
title: 使用 ArchInstall 安装 Arch Linux
description: 在2024年5月版的 Arch Linux 安装镜像的基础上，使用 ArchInstall 安装系统全过程。
date: 2023-05-23 20:37:07
updated: 2024-05-12 20:54:15
image: https://7.isyangs.cn/24/66640095c770c-24.jpg
cover: https://7.isyangs.cn/24/66640095c770c-24.jpg
banner: https://7.isyangs.cn/24/66640095c770c-24.jpg
categories: [经验分享]
tags: [教程, archlinux, 系统]
sticky: 20
---

## 安装前准备

安装 Arch Linux 时务必小心！如果误操作可能会导致 Windows 系统无法启动。请留好 {% post_link 2023/windows-setup-guide %} 和 {% post_link 2024/archlinux-boot-repair %}，以备后日之用。

{% grid %}
<!-- cell -->
{% banner
   bg:https://7.isyangs.cn/24/65b66289c9a2e-24.jpg
   link:/2023/windows-setup-guide
 %}
{% endbanner %}
<!-- cell -->
{% banner
   bg:https://7.isyangs.cn/24/66640097e0cbb-24.jpg
   link:/2024/archlinux-boot-repair
 %}
{% endbanner %}
{% endgrid %}

### 制作安装镜像

#### 下载镜像

官方提供了 [Arch Linux 下载页面](https://archlinux.org/download/)，但是推荐在镜像站下载安装镜像，速度更快：

- [北京外国语大学开源软件镜像站](https://mirrors.bfsu.edu.cn/archlinux/iso/)
- [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/)
- [南京大学开源软件镜像站](https://mirror.nju.edu.cn/archlinux/iso/)

#### 装载到U盘

可以使用 [Rufus](https://rufus.ie/zh/) 或 [balenaEtcher](https://etcher.balena.io/) 等类似工具把镜像刻录到 U 盘。但我个人更推荐先给 U 盘安装 [Ventoy](https://www.ventoy.net/)，只要把镜像文件放入 U 盘就可以在启动时手动选择，并且不影响 U 盘正式使用。

### 关闭安全启动

Ventoy 需要加载自定义证书文件才能在启用“安全启动”特性的设备上正常启动，一些 Linux 安装环境可能也不支持“安全启动”。方便起见，可以在 BIOS 设置里关闭“安全启动”特性，装好系统后再打开。

### 联网

{% copy ping -c 4 1.1.1.1 prefix:# %}
测试网络，如果无报错，可以跳过此节。

#### 通过数据线共享手机网络

> 有时买到的第三方 USB 线只能充电，无法传输数据。因此建议用手机原装充电器（其他手机的也可）提供的数据线来共享手机网络。
>
> 如果通过数据线共享手机网络，建议同时连接 WiFi，防止意外断联导致安装失败。

使用数据线连接手机与电脑，在手机热点设置中启用“USB共享网络”即可。注意，如果你重启了电脑或者断开了数据线连接，可能需要在手机设置中重启（关闭并打开）USB 共享网络功能。

#### 连接WiFi

- 部分电脑会限制无线网卡，故建议先执行命令解锁 WiFi
  {% copy rfkill unblock wifi prefix:# %}
- 进入联网工具
  {% copy iwctl prefix:# %}
- 扫描 WiFi
  {% copy station wlan0 scan prefix:[iwd]# %}
- 查看 WiFi 列表
  {% copy station wlan0 get-networks prefix:[iwd]# %}
- 连接 WiFi
  {% copy station wlan0 connect [SSID] prefix:[iwd]# %}
  - 可以按 Tab 键补全 SSID（WiFi 名称）。
  - 安装镜像的 TTY 终端没有中文字体，不支持中文显示，所以用户可能无法区分中文名称的 WiFi。
- 退出 iwctl
  {% copy exit prefix:[iwd]# %}

### 检查硬盘分区

如果你的电脑上安装了 Windows，建议提前在 Windows 中留出安装系统的硬盘分区，你也可以使用 PE 系统来管理硬盘分区。

- 列出硬盘分区
  {% copy lsblk -o+LABEL,FSTYPE prefix:# %}
- 若有必要，创建新的硬盘分区
  {% copy cfdisk /dev/[driver] prefix:# %}
  - Driver（硬盘驱动器）名称应当类似于 sda、nvme0n1 等。
- 如有必要，格式化分区
  {% copy mkfs.btrfs /dev/[partition] prefix:# %}
  - 如果格式化失败，确认不含重要数据后，加上 `-f` 选项强制格式化。

## 换源

使用 Reflector 更新源，可以使安装速度更快。

{% copy reflector --verbose --country China --sort rate --save /etc/pacman.d/mirrorlist prefix:# %}

换源后，请**不要**在 ArchInstall 中设置 Mirrors 选项。

## 使用 ArchInstall 安装

- 进入安装工具
  {% copy archinstall prefix:# %}
  - 如果等待时间过长，可以按 Ctrl-C 组合键中断，然后重新执行命令。
  - 如果多次尝试都不能进入安装界面，那么建议更换网络环境再试。

### 选择镜像下载地区

> 如果你使用过 Reflector 更新源，那么**不应该**进行此项设置。
>
> 你可以通过在 Mirrors 选项内按 Ctrl-C 重置设置。

移动到`Mirrors`项敲回车，键入`/ch`搜索，选择`China`。

### 选择本地语言

进入 `Locales` 选项：

- `Locale language` 选择 `zh_CN`。
- 其他选项保持默认值，即可 `← Back`。

### 选择硬盘分区

进入 `Disk configuration` 选项：

- 选择 `Use a best-effort default partition layout` 会清空硬盘并自动规划分区。
- 选择 `Manual Partitioning` 项可以手动配置分区。
  - 选择的设备（Device）应该是你的硬盘。
  - 随后选择 EFI 启动分区，指定挂载点（Mount Point）为 `/boot`（更规范的配置是 `/boot/efi`，但可能会安装失败）。可以选择 Windows 的引导分区作为启动分区，这个分区一般有以下特征：
    - 在第一个硬盘的第一个分区（`/dev/nvme0n1p1`）。
    - 大小为 260MB。
    - 文件系统为 FAT32。
  - 选择新创建的 BtrFS 分区，指定挂载点为 `/`。
    - 新的 ArchInstall 可能需要格式化（Wipe data/Format）这个分区。
    - 如果无法直接设置挂载点，可能要设置一个子卷（Subvolume）`@`，挂载点为 `/`，再`Confirm and exit`。
  - 可以按照需求添加其他分区，随后在硬盘主菜单 `Confirm and exit`。

### 选择引导工具

`Bootloader` 选择使用默认的 `Systemd-boot` 即可，一般不需要设置。

如果你的电脑开启了独显直连功能，使用 Grub 会导致操作视图更新卡顿。

### 设置主机名

`Hostname` 选项中可以设置主机名。注意，主机名不是用户名。

### 设置 Root 用户密码

在 `Root password` 选项中设置 Root 用户密码即可启用 Root 用户。密码需要输入两次才会确定。

### 添加用户

`User account` 选择中，选择 `Add a user`，输入用户名、两遍密码，并且确认作为 `Superuser`（超级用户，即具有 sudo 权限），然后选择 `Confirm and exit`。

### 选择用户环境

在 `Profile` 选项中：

- `Type` 选择桌面环境（`Desktop`），桌面环境推荐 `KDE`。
- `Graphics driver`：如果是双显卡笔记本，可以先装 CPU 对应的开源核显驱动（AMD 或者 Intel），系统安装完成后再安装独显驱动。
- 其他选项默认即可，选择 `← Back`。

### 选择音频驱动

`Audio` 选项中，默认的 `Pipewire` 即可，`PulseAudio` 比较老旧，建议仅在 Pipewire 失效时选择。

### 可选包

`Additional packages` 选项中，可以添加以下包，用空格作为分隔符，回车确认：

- `nvidia`：Nvidia 专有的显卡驱动。
- `ttf-sarasa-gothic`：更纱黑体，体积较大但显示效果更好，以防重启之后中文显示为方框。

### 设置网络

`Network configuration` 选项选择 `Use NetworkManager` 即可。

### 选择时区

`Timezone` 选项中，键入 `/shang` 可以搜索，选择 `Asia/Shanghai`。

## 安装

移动到 `Install` 项敲回车，确认无误后再敲回车，等待即可。

安装成功后，会进入新系统的 Chroot 环境，你可以自由安装软件，也可输入 `exit` 退出。我推荐按 `Ctrl-Alt-Delete` 直接重启系统。

## 进入系统

如果你是 Nvidia 显卡用户，在一会的 SDDM 登录界面中，左上角的 Session 菜单中应当选择 `Plasma(X11)` 选项，而不是 Wayland 环境。

可以参照 {% post_link 2023/archlinux-configure %} 进行后续配置。

::md-link-card
---
icon: https://7.isyangs.cn/24/6664009851eb0-24.jpg
title: Arch Linux 初步配置
link: /2023/archlinux-configure
---
::