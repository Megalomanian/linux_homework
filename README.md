# Linux 期末大作业

作者：计算机科学与技术 三班 朱林立

[GitHub 主页](https://github.com/Megalomanian)，给个 star 呗

虚拟机文件`LinuxHOMEWORK111.ova`以 ova 格式导出，如需导入 virtual box 或 vm ware 中检查，可[参考此文章](https://blog.csdn.net/weixin_43031313/article/details/129897809)

## 题目 1：在 virtual box 中安装 archlinux

从[此链接](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/archlinux-2025.11.01-x86_64.iso)下载 archlinux 的 iso 并导入

系统配置 CPU、内存、显存、硬盘如下

![image.png](image.png)

![image.png](image%201.png)

![image.png](image%202.png)

ping 检查网络，同步时间

```bash
ping www.baidu.com -c 3
timedatectl set-ntp true
```

分区

```plaintext
cfdisk /dev/sda
```

![image.png](image%203.png)

配置镜像源

注：本文所有文本编辑内容使用 vim 编辑器，vim 编辑器入门可参考以下视频：[视频链接](https://www.bilibili.com/video/BV13t4y1t7Wg)

```plaintext
vim /etc/pacman.d/mirrorlist
```

dd 剪切清华源，p 黏贴到第一行

![image.png](image%204.png)

安装基础系统里要用的东西（如果配置了镜像源就可以极大加速下载环节）

```plaintext
pacstrap /mnt base linux linux-firmware vim networkmanager sudo zsh
```

![image.png](image%205.png)

生成 fstab，进系统

```plaintext
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab   # 这条命令用于确认分区
arch-chroot /mnt
```

![image.png](image%206.png)

## 题目 2：设置 hostname 为学号

```plaintext
echo 12024242193 > /etc/hostname
```

vim 编辑 hosts 文件

```plaintext
vim /etc/hosts
```

写入以下内容

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   12024242193.localdomain 12024242193
```

[https://www.notion.so](https://www.notion.so)

## 题目 3：设置时区

```plaintext
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc       # 生成 /etc/adjtime
```

## 题目 4：创建普通用户 + wheel 组 + sudo

设置 root 密码：

```plaintext
passwd
```

创建普通 student 用户，设置密码：

```plaintext
useradd -m -G wheel -s /bin/bash student
passwd student
```

允许 wheel 用户组使用 sudo：

```plaintext
EDITOR=vim visudo
```

![image.png](image%207.png)

## 题目 5：把普通用户的 shell 切换到 zsh

1. 修改用户 shell（之前装过 zsh 了）：

```bash
chsh -s /bin/zsh student
```

![image.png](image%208.png)

## 题目 1（后半部分）：此时安装引导（GRUB，BIOS + MBR）

```bash
pacman -S grub
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

![image.png](image%209.png)

## 题目 6：安装图形界面 + 开机启动（5 分）

### 1. 安装 X 相关

```bash
pacman -S xorg-server xorg-apps xorg-xinit
```

### 2. 安装 XFCE（这个桌面环境我用着比较习惯）

```bash
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
```

### 3. 开机自启图形登录管理器（以 lightdm 为例）

```bash
systemctl enable lightdm
```

重启一下进入图形化界面

![image.png](image%2010.png)

## 题目 7：安装浏览器（谷歌或者火狐，我这块都装了）

```bash
pacman -S firefox # 装火狐
pacman -S chromium # 装谷歌
```

![image.png](image%2011.png)

![image.png](image%2012.png)

## 题目 8：设置中文字体

改/etc/locale.gen，取消这两行的注释：

![image.png](image%2013.png)

![image.png](image%2014.png)

设置默认语言：

```plaintext
echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```

装中文字体：

```plaintext
pacman -S noto-fonts-cjk wqy-microhei
```

![image.png](image%2015.png)

重新生成本地信息：

![image.png](image%2016.png)

reboot 重启：

![image.png](image%2017.png)

## 题目 9：安装输入法，开机自启

```plaintext
pacman -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-qt
```

改/etc/environments

![image.png](image%2018.png)

重启测试输入法

![image.png](image%2019.png)

## 题目 10：安装 nginx 设置端口 8080 开机自启

安装

```plaintext
pacman -S nginx
```

改配置文件/etc/nginx/nginx.conf

![image.png](image%2020.png)

设置开机自启

```plaintext
systemctl enable --now nginx
```

![image.png](image%2021.png)

## 题目 11：安装 sshd，端口 2222，开机启动

运行命令：

```plaintext
pacman -S openssh
vim /etc/ssh/sshd_config

#将Port 22改2222
```

![image.png](image%2022.png)

启动并配置开机自启

```plaintext
systemctl enable --now sshd
```

运行以下命令，从本地连接测试：

```plaintext
ssh student@127.0.0.1 -p 2222
```

出现以下内容代表成功

![image.png](image%2023.png)

## 题目 12：system_info 脚本

将以下内容写入/usr/local/bin/system_info

```plaintext
#!/usr/bin/env bash

USER_NAME=$(whoami)

CURRENT_SHELL="$SHELL"

MEM_USAGE=$(free | awk '/Mem:/ {printf("%.2f%%", $3/$2*100)}')

DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')

echo "Current user: ${USER_NAME}"
echo "Current shell: ${CURRENT_SHELL}"
echo "Memory usage: ${MEM_USAGE}"
echo "Disk usage (/): ${DISK_USAGE}"
```

赋予执行权限，运行

![image.png](image%2024.png)
