# PXE Install ubuntu Desktop 2204
## Prepare
---------------------------------
基础：网络内的 PXE server & clinets 网络在一个网段
   
0 . [autoinstall over PXE（最重要介绍了installer）](https://www.molnar-peter.hu/en/ubuntu-jammy-netinstall-pxe.html)  
1 . [官方netinstall文档（for server）](https://ubuntu.com/server/docs/install/netboot-amd64)  
2 . [preseed和内核启动参数参考,有价值](https://zhangguanzhang.github.io/2019/08/06/preseed/)  
3 . [基于Ubuntu 20.04 Server搭建PXE自动安装环境](https://segmentfault.com/a/1190000040527863?utm_source=sf-similar-article)  
4 . [基于Ubuntu server的 PXE网络装机](https://juejin.cn/post/7198336167913439291)  
5 . https://blog.csdn.net/u010438035/article/details/128396790  （虚拟机）  
6 . https://www.cpweb.top/1698  
7 . https://blog.gpx.moe/2023/02/09/pxe-server/  pxe服务器搭建较为详细  
8 . [批量安装Windows系统](https://www.modb.pro/db/227748)  
9 . [官方讨论帖](https://discourse.ubuntu.com/t/netbooting-the-live-server-installer/14510/15)  
10 . [Ubuntu16.04桌面版pxe启动实现自动安装（原型）](https://blog.csdn.net/wanpengpenga/article/details/81233803?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-81233803-blog-102515747.235%5Ev38%5Epc_relevant_sort_base2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-81233803-blog-102515747.235%5Ev38%5Epc_relevant_sort_base2&utm_relevant_index=2)  
11 . [PXE排错参考,重要](https://zhuanlan.zhihu.com/p/67192427)

## 执行过程
-------------------------------------------
### 一、 开启PXE启动程序（F9搜索）(本案例使用的华硕主板)
---------------------------------------
	0. uefi（实验不成功，本方案不适用）
	1. 高级--网络堆栈--enable--pxe ipv4 enable 
    2. 启动--CSM--启动设备控制（仅 UEFI）--从网络设备启动（仅UEFI）
    3. 启动项为pxe ipv4

UEFI启动才会请求bootx64.efi，如果是传统启动模式（Legacy），那么PXE Client会请求pxelinux.0

	1. legacy （本方案适用）
    1. 高级--网络堆栈--enable--pxe ipv4 enable
    2. 启动--CSM--启动设备控制（仅legacy）--从网络设备启动（仅legacy）--打开legacy oprom
    3. 启动项为realtek...
计算机 S/C 直连

### 二、 依赖软件
----------------------------------------------
#### 1 . 安装
```
sudo apt-get install -y apache2  nfs-kernel-server isc-dhcp-server tftpd-hpa  
```

| 服务 | 作用 |
| :---: | :---: |
| DHCP | 分配ip，联入局域网 |
|TFTP|传输内核文件、pxe引导文件|
|NFS|传输镜像|
|HTTP|本例用于传输preseed文件|


#### 2 . DHCP服务排错
	2.1  机器联入局域网，在办公司交换机直连后成功
	2.2  windows系统请求时，dhcp服务正常日志（作为参考的对照）
```
Jun 19 16:05:38 inboc-sys-lc dhcpd[1517378]: DHCPDISCOVER from 58:11:22:bd:27:35 via enp7s0
Jun 19 16:05:38 inboc-sys-lc dhcpd[1517378]: DHCPOFFER on 10.10.6.3 to 58:11:22:bd:27:35 (PC-202306051659) via enp7s0
Jun 19 16:05:38 inboc-sys-lc dhcpd[1517378]: DHCPREQUEST for 10.10.6.3 (10.10.6.1) from 58:11:22:bd:27:35 (PC-202306051659) via enp7s0
Jun 19 16:05:38 inboc-sys-lc dhcpd[1517378]: DHCPACK on 10.10.6.3 to 58:11:22:bd:27:35 (PC-202306051659) via enp7s0
```

	错误：pxe启动  dhcp--没有request请求，客户端输出 PXE-E16 NO valid offer recieved
	***原因：主板PXE设置不正确，需要全设置为仅uefi  或者  仅legacy***
```
Jun 19 16:19:26 inboc-sys-lc dhcpd[1517378]: DHCPDISCOVER from 58:11:22:bd:27:35 via enp7s0
Jun 19 16:19:27 inboc-sys-lc dhcpd[1517378]: DHCPOFFER on 10.10.6.4 to 58:11:22:bd:27:35 via enp7s0
```

#### 3. TFTP 服务排错
	3.1
PXE-T02：only absolute filenames allowed
PXE-E3C: TFTP error – Access violation
pxe tftp open time out
```
配置文件添加
FTP_OPTIONS="--secure"
RUN_DAEMON="yes"
```

	 3.2
 tftp: client does not accept options
 pxe file 0 bytes
```
不影响过程，可以忽略，实际可以读取到
```

2 . 准备系统加载程序
```
cd PXE/syslinux/
wget -c https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.zip
unzip syslinux-6.03.zip -d /path

cp ./bios/core/pxelinux.0  /home/chao.long/pxe/tftp
cp ./bios/com32/elflink/ldlinux/ldlinux.c32 ../tftp/
cp ./bios/com32/lib/libcom32.c32 /home/chao.long/pxe/tftp  
cp ./bios/com32/libutil/libutil.c32  /home/chao.long/pxe/tftp  
cp ./bios/com32/menu/vesamenu.c32 ../tftp/    
```


#### 4. DHCP 配置 
```
cat /etc/dhcp/dhcpd.conf

	default-lease-time 600;
	max-lease-time 7200;
	ddns-update-style none;
	allow booting;
	allow bootp;
	
	subnet 10.10.6.0 netmask 255.255.255.0 {
	  range 10.10.6.1 10.10.6.20;
	  option routers 10.10.6.254;
	  option broadcast-address 10.10.6.255;
	  next-server 10.10.6.1;   #指定TFTP服务器
	  filename "pxelinux.0";   #指定下载，放置在TFTP 根目录
	  interface "enp7s0";
	}
```


#### 5. TFTP 配置
5.1
```
cat /etc/default/tftpd-hpa
	TFTP_USERNAME="tftp"
	TFTP_DIRECTORY="/path/tftp"
	TFTP_PORT="69"
	TFTP_ADDRESS="0.0.0.0:69"
	TFTP_OPTIONS="--secure"
	RUN_DAEMON="yes"

systemctl restart tftpd-hpa
检查：可以找一个客户端执行 tftp server_ip get file，查看是否下载
```

5.2 目录结构
```
/path/tftp
├── casper
│   ├── initrd      #initrd（initial ramdisk）是一个临时的根文件系统，它在内核启动时被加载到内存中。initrd包含操作系统启动所需的所有必要文件和驱动程序,在根文件系统正常挂载之前，暂时将initrd文件系统作为根文件系统
│   └── vmlinuz     #vmlinuz 是 Linux 内核文件的压缩版本，包含操作系统内核的所有代码和相关的驱动程序。当计算机启动时，引导程序会加载 vmlinuz 文件，并将控制权交给内核，从而启动 Linux 操作系统
├── ldlinux.c32     #提供了 PXELinux 引导程序的基本功能。它包含了加载配置文件、解析指令、加载内核等功能
├── libcom32.c32    #提供了一些常用的库函数，例如字符串操作、文件操作等。
├── libutil.c32     #提供了一些常用的工具函数，例如时间处理、网络操作等
├── pxelinux.0      #PXELinux 引导程序的核心文件，是在网络上启动计算机并加载操作系统时必需的文件,通过 DHCP 服务器指定计算机在启动时从哪个 TFTP 服务器上下载该文件     
├── pxelinux.cfg
│   ├── default     #`PXELinux`引导程序的配置文件，程序启动时，它会尝试加载该文件作为默认配置文件，以确定应该如何启动计算机并加载操作系统
│   └── pxe.conf    #可以不准备，对选择内核时的图形化界面做设置
└── vesamenu.c32    #提供了一个图形化的菜单界面，方便用户选择启动菜单中的操作系统或其他选项
```

	5.3 以上文件来源
		5.3.1
```
wget  https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.tar.gz
解压、查找、拷贝，./bios目录下的以下文件，在本方案有效
0. pxelinux.0  - PXELinux引导程序的核心文件
1. vesamenu.c32 - 用于提供简单的菜单界面 
2. ldlinux.c32 - PXE链引导文件
3. libcom32.c32 - PXE C接口库
4. libutil.c32 - PXE实用工具库
```

		5.3.2 后台下载，任意路径
```
wget https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-desktop-amd64.iso & 
sudo mount ./ubuntu-22.04.2-desktop-amd64.iso   /path/nfs
	mount: /media: WARNING: device write-protected, mounted read-only.(系统正常输出)
cp /path/nfs/casper/initrd   /path/tftp/casper
cp /path/nfs/casper/vmlinuz  /path/tftp/casper
```

		5.3.3 pxelinux.cfg/default
```
default vesamenu.c32
prompt 0
timeout 30
ontimeout auto
menu title PXE Automatic Install
label auto
        menu label Ubuntu Desktop 22.04 Automatic Installer
        menu default
        kernel casper/vmlinuz
        initrd casper/initrd
        append automatic-ubiquity ip=dhcp netboot=nfs nfsroot=10.10.6.1:/home/inboc/pxe/tftp/iso  url=http://10.10.6.1/preseed.seed

成功下载iso，但是 can't find live filesystem： （此书写方式实验不成功，应用上一种）
        append  ip=dhcp auto=true priority=critical url=http://10.10.6.1/iso/ubuntu-20.04.5-desktop-amd64.iso preseed/url=http://10.10.6.1/ubuntu.seed

```
#### 6. NFS配置
	6.1 目录结构
```
/path/nfs
├── boot
├── boot.catalog
├── casper
├── dists
├── EFI
├── install
├── md5sum.txt
├── pool
|__ .disk
└── preseed
```

	6.2 内容准备
```
sudo mount ./ubuntu-22.04.2-desktop-amd64.iso   /path/nfs
或者
挂载到另一个位置，例如/mnt
sudo cp -r  /mnt/*  /path/nfs
sudo cp -r  /mnt/.disk  /path/nfs
```

	6.3  共享
```
sudo vim /etc/exports
  /home/inboc/pxe/tftp/iso  10.10.6.0/24(ro,no_subtree_check)
```

	6.4  修改配置，使支持tcp
```
sudo vim /etc/default/nfs-kernel-server
	RPCNFSDCOUNT=8
	RPCNFSDPRIORITY=0
	RPCMOUNTDOPTS="--manage-gids"  
	RPCNFSDOPTS="-T tcp"  #支持tcp
	NEED_SVCGSSD=""
	RPCSVCGSSDOPTS=""

sudo systemctl resart nfs-kernel-server
```

#### 7. HTTP配置
	7.1  目录结构
```
/var/www/html/
└── preseed.seed
```

>7.2 preseed.seed 文件
> 	[preseed 模板](https://www.debian.org/releases/stable/example-preseed.txt)
> 	[官方autoinstall-generator介绍](https://discourse.ubuntu.com/t/autoinstall-generator-tool-to-help-with-creation-of-autoinstall-files-based-on-preseed/21334)（使用后输出报错，无解决方案）
> 	[cloud_init文档](https://cloudinit.readthedocs.io/en/latest/index.html)
> 	[preseed中解决update & software](https://askubuntu.com/questions/1441386/updates-and-other-software-in-preseed-ubuntu-22-04)

##### 有效版：
```
# Ubuntu ubiquity seed file
# Set up the installation language, country and locale.
ubiquity debian-installer/locale string en_US.UTF-8

# Set up the keyboard layout.
ubiquity debian-installer/language string en
ubiquity debian-installer/country string US
ubiquity keyboard-configuration/layoutcode string us

#update & software
ubiquity ubiquity/minimal_install boolean true
ubiquity apt-setup/restricted boolean false
tasksel tasksel/first multiselect
ubiquity ubiquity/use_nonfree boolean true
ubiquity ubiquity/restricted-addons boolean false
ubiquity ubiquity/download_updates boolean false

# Partition the disk.
ubiquity partman-auto/init_automatically_partition select Guided - use entire disk 
ubiquity partman-auto/method string regular 
ubiquity partman-auto/choose_recipe select atomic
ubiquity partman/confirm_write_new_label boolean true
ubiquity partman/confirm boolean true
ubiquity partman/confirm_nooverwrite boolean true
ubiquity partman/choose_partition select finish
# Create an initial user.
ubiquity passwd/user-fullname string inboc
ubiquity passwd/username string inboc
ubiquity passwd/user-password password Inboc@2020
ubiquity passwd/user-password-again password Inboc@2020
ubiquity passwd/user-uid string 1000

# Set up the time zone.
ubiquity time/zone string Asia/Shanghai

ubiquity grub-installer/grub2_instead_of_grub_legacy boolean true
ubiquity grub-installer/only_debian boolean true
ubiquity finish-install/reboot_in_progress note
# Specify a command to run after a successful installation.
ubiquity ubiquity/success_command string apt-get install -y openssh-server
# reboot after installed
ubiquity ubiquity/reboot boolean true

```

##### 第三版：
```
#cloud-config
autoinstall:
  identity:
    hostname: jammy-desktop
    password: $6$5lpwCLsKLEzMkSJc$keOAhA6aO/5RocGThmhVA7LSNuW911Rx5HHXFEa75oGK20cEdAAgn14H5f5nGeq6QgcSyLPrWcg1.JvjXbhrN/
    realname: Ubuntu user
    username: ubuntu
  keyboard:
    layout: hu
    toggle: null
    variant: ''
  locale: hu_HU.UTF-8
  storage:
    config:
# Partition table
    - { ptable: gpt, path: /dev/vda, wipe: superblock, preserve: false, name: '', grub_device: false, type: disk, id: disk-vda }
# EFI boot partition
    - { device: disk-vda, size: 536870912, wipe: superblock, flag: boot, number: 1, preserve: false, grub_device: true, type: partition, id: partition-0 }
    - { fstype: fat32, volume: partition-0, preserve: false, type: format, id: format-0 }
# Linux boot partition
    - { device: disk-vda, size: 1073741824, wipe: superblock, flag: '', number: 2, preserve: false, grub_device: false, type: partition, id: partition-1 }
    - { fstype: ext4, volume: partition-1, preserve: false, type: format, id: format-1 }
# Partition for LVM, VG
    - { device: disk-vda, size: -1, wipe: superblock, flag: '', number: 3, preserve: false, grub_device: false, type: partition, id: partition-2 }
    - { name: ubuntu-vg, devices: [ partition-2 ], preserve: false, type: lvm_volgroup, id: lvm_volgroup-0 }
# LV for root
    - { name: ubuntu-lv, volgroup: lvm_volgroup-0, size: -1, wipe: superblock, preserve: false, type: lvm_partition, id: lvm_partition-0 }
    - { fstype: ext4, volume: lvm_partition-0, preserve: false, type: format, id: format-2 }
# Mount points
    - { path: /, device: format-2, type: mount, id: mount-2 }
    - { path: /boot, device: format-1, type: mount, id: mount-1 }
    - { path: /boot/efi, device: format-0, type: mount, id: mount-0 }
# Swapfile on root volume
    swap:
      swap: 1G
  late-commands:
  - 'echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /target/etc/sudoers.d/ubuntu-nopw'
  - chmod 440 /target/etc/sudoers.d/ubuntu-nopw
  - curtin in-target --target=/target -- sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"/' /etc/default/grub
  - curtin in-target --target=/target -- apt-get install -y ubuntu-desktop plymouth-theme-ubuntu-logo grub-gfxpayload-lists
  version: 1
```

##### 第二版：
```
# Ubuntu ubiquity seed file

# Enable extras.ubuntu.com. 
ubiquity ubiquity/enable_extras boolean true 

# Install the Ubuntu desktop.
ubiquity tasksel/first multiselect ubuntu-desktop


# Set up the installation language, country and locale.
ubiquity debian-installer/locale string en_US.UTF-8
ubiquity debian-installer/language string en
ubiquity debian-installer/country string CN

# Set up the keyboard layout.
ubiquity console-setup/layoutcode string us
ubiquity console-setup/layout select U.S. English
ubiquity console-setup/variantcode select U.S. English
ubiquity console-setup/codeset select . Combined - Latin; Slavic Cyrillic; Hebrew; basic Arabic；UTF-8
# wifi
ubiquity netcfg/enable boolean false preseed
# Partition the disk.
ubiquity partman-auto/init_automatically_partition select Guided - use entire disk 
ubiquity partman-auto/method string regular 
ubiquity partman-auto/choose_recipe select All files in one partition (recommended for new users)
ubiquity partman/confirm_write_new_label boolean true
ubiquity partman/choose_partition select Finish partitioning and write changes to disk
ubiquity partman/confirm boolean true
ubiquity partman/confirm_nooverwrite boolean true

# Create an initial user.
ubiquity passwd/user-fullname string inboc
ubiquity passwd/username string inboc
ubiquity passwd/user-password-again password Inboc@2020
ubiquity user-setup/encrypt-home boolean false
# Set up the time zone.
ubiquity time/zone string Asia/Shanghai
# Reboot the system when the installation is complete.
ubiquity ubiquity/reboot boolean true  

# Specify a command to run after a successful installation.
ubiquity ubiquity/success_command string apt-get install -y openssh-server

```

##### 第1版（ debian installer）：
```
# Newer ubiquity command
ubiquity partman-auto/disk string /dev/sda
ubiquity partman-auto/method string regular
ubiquity partman-lvm/device_remove_lvm boolean true
ubiquity partman-md/device_remove_md boolean true
ubiquity partman-auto/choose_recipe select atomic

# This makes partman automatically partition without confirmation
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Locale
d-i debian-installer/locale string en_US
d-i console-setup/ask_detect boolean false
d-i console-setup/layoutcode string us

# Network
#d-i netcfg/get_hostname string $HOST
#d-i netcfg/get_domain string localdomain
d-i netcfg/choose_interface select auto

# Clock
d-i clock-setup/utc-auto boolean true
d-i clock-setup/utc boolean true
d-i time/zone string Asia/Shanghai
d-i clock-setup/ntp boolean true

# Packages, Mirrors, Image
d-i mirror/country string CN
d-i apt-setup/multiverse boolean true
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true

# Users
d-i passwd/user-fullname string inboc
d-i passwd/username string inboc
d-i passwd/user-password-crypted password f0b8992a4171a4d75756035300ff837e0755218d440e6db2a4723720911cdf5efd9a4b29dbacef78934e51cca4b481969d328536d78056b9e46fcc5902625b6e
d-i passwd/root-login boolean false
# d-i passwd/root-password-crypted password rootEncryptedPasswd
d-i user-setup/allow-password-weak boolean true

# Grub
d-i grub-installer/grub2_instead_of_grub_legacy boolean true
d-i grub-installer/only_debian boolean true
d-i finish-install/reboot_in_progress note

```

#### 8. 当前问题
	偶尔关机重启会掉显卡驱动,手动重装
    对 uefi、legacy 模式认识区分不足