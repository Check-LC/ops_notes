2023-08

0 . [Guides](https://www.youtube.com/watch?v=ZabwkpdaBK0)
    [Video](https://www.youtube.com/watch?v=pSC2-45TQug)

1 . 开机，Volume up 键进入 bios，关闭 Security-secure boot，勾选 boot-usb storage，左滑该选项即可引导安装

2 . Ventory GPT 格式安装到 U 盘
	镜像：ubuntu-desktop-2204

3 . 需求：解决键盘和触摸屏的问题

4 . [安装新内核6.1+，支持 surface 硬件](https://blog.csdn.net/yaxuan88521/article/details/128924018)
	sudo add-apt-repository ppa:cappelikan/ppa
	sudo apt update
	sudo apt install -y mainline
	安装6.4的内核，在后续编译中会失败 (Make Error 2 Error10 Error 127)
	安装6.1.0，后续编译以及使用正常

5 . 触摸屏模块编译
	[guidances partion: Setting up the Touch Screeen](https://www.linux.org/threads/ubuntu-22-04-on-surface-pro-7.43071/)
5.1 编译安装 ithc，实现单点触控 [repo: https://github.com/quo/ithc-linux](https://github.com/quo/ithc-linux) 
```
	sudo apt install git meson build-essential dkms pkg-config cmake systemd -y
	cd Dowloads
	git clone https://github.com/quo/ithc-linux.git
	cd ithc-linux
	sudo make dkms-install
	sudo modprobe ithc
```
5.2  修改引导文件 
```
	sudo vim  /etc/default/grub
		 GRUB_CMDLINE_LINUX_DEFAULT="quiet intremap=nosid splash" 
	sudo update-grub
```

6 . 编译安装 iptsd，实现多点触控和触控笔 [repo: https://github.com/linux-surface/iptsd](https://github.com/linux-surface/iptsd)
```
sudo apt install -y gcc g++ ninja-build libeigen3-dev libfmt-dev libinih-dev libgsl-dev
git clone https://github.com/linux-surface/iptsd
cd iptsd
meson build
ninja -C build
sudo ninja -C build install
```

- 检查是否由 systemd 托管,正常情况下已经存在此文件，/lib/systemd/system/iptsd.service
```
systemctl status iptsd
```

7 . 系统应能正常使用

8 . 重新安装为windows（非官方恢复）
	8.1 镜像来源[uupdump](https://www.uupdump.cn/download.php?id=6661ce49-e38c-49ac-b4bf-d13779b8eed6&pack=en-us&edition=professional), 执行linux命名的 sh 脚本; 在此之前阅读 readme 文件，安装包 " ***sudo apt install aria2 cabextract wimtools chntpw genisoimage*** "
	8.2 key：NMF2M-4HB3F-D63KV-7PPDB-WHV26
	8.3 跳过激活，使用本地账户。shift + F10 , cmd 执行 "***oobe\bypassnro***""
	8.4 驱动补充安装,地址：[官方驱动下载](https://www.microsoft.com/en-us/download/details.aspx?id=104680)，方法：双击执行安装。
	8.5 重启，应可正常使用

9 . 官方恢复镜像
 通过序列号匹配镜像文件，写入U盘启动器
 将 home 升级到 pro 版本系统
