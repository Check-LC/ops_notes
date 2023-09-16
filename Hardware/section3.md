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