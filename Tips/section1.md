0 . mark
	[Devops Roadmap](https://gitbook.curiouser.top/)
	[咖啡吧](https://ops-coffee.cn/knowledge)
	[优点知识](https://youdianzhishi.com)
	[云原生资料库 (jimmysong.io)](https://lib.jimmysong.io/)
	[某集体站点](https://www.infvie.com/)
	[仲儿的自留地](https://lisz.me/)
	[devops](https://owen2016.gitee.io)
	[超详细站点](https://wiki.shileizcc.com/confluence/#all-updates)
	[ansible 中文指南](https://ansible-tran.readthedocs.io/en/latest/docs/playbooks.html)


1 . mount 挂载指定域下的Samba服务器到本地
```
sudo mount -t cifs   //10.13.1.5/soft  /home/chao.long/samba  -o  domain=inboc,username=chao.long,password=****,uid=********,gid=****
```

2 . 应用许可
```
intellj.idea  
许可证通过passwordsafe中文件查看账户和密码，bitwarden有存
```

3 . docker 镜像查找和拉取 
```
docker search repo.inboc.net/inboc/conda:

docker pull ghcr.io/ldapaccountmanager/lam:8.4@sha256:283726bd23510f1c3bfbdcbfe861e6599e070616543aed02e9756075c97a9938
```

4 . docker 在普通用户下不能使用

- 报错示例
```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
```

- 解决办法
```
ll /var/run/docker.sock
sudo usermod -aG docker inboc
sudo systemctl restart docker
sudo -i 重新登录使生效 
su - inboc
```

5 . 向一个没有网络的机器安装某个包，解决其安装和相关依赖的办法  
apt-redepends 可以下载软件包和相关依赖用于离线安装  
[apt-rdepends忽略部分包的方法](https://superuser.com/questions/1112525/ignore-apt-get-download-errors/1137335#1137335)

6 . linux 对容器抓包
```
找到Pid
	docker top <container_id/_name> 
	docker inspect <container_id/_name> | grep Pid
进入容器名称空间
	sudo nsenter -t <Pid> -n
	exit 退出
在其中抓包
	tcpdump -i <net device>
k8s环境,需要定义节点,在相应节点找到container_id,同样执行抓包
	kubectl get pod -n <命名空间> <pod名> -o wide
	kubectl get pod -n <命名空间> <pod名> -o yaml | grep containerID
```

7 . ubuntu 2204 登录帐号后黑屏（本例是由于输入法安装导致的）
```
密码登录后 ctrl+alt+F3 进入命令行
sudo systemctl stop gdm3
进入当前用户的家目录执行：  mv .config .cofig.bak
sudo systemctl start gdm3
重启电脑，恢复正常
```

8 . ubuntu对屏幕拓展和设置主副屏
```
xrandr  # 查看输出接口
xrandr --output HDMI-0 --auto --primary   # 将hdmi输出的屏幕设置为主屏
xrandr --output DP-3 --right-of HDMI-0 --auto  # Hdmi输出的屏幕设置为右侧屏幕
```

9 .  k8s 集群， kubeadm 部署疑问
- 9.1
```
sudo kubectl apply -f kube-flannel.yml 
    The connection to the server localhost:8080 was refused - did you specify the right host or port?

（主从节点一致，只要是希望运行这个命令，即检查以下内容）
kubectl命令需要使用kubernetes-admin来运行，需要admin.conf文件（conf文件是通过“ kubeadmin init”命令在主节点/etc/kubernetes 中创建），但是从节点没有conf文件，也没有设置 KUBECONFIG =/root/admin.conf环境变量，所以需要复制conf文件到从节点，并设置环境变量就OK了
解决办法：
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- 9.2
```
kubectl describe nodes inboc-sys-test-09
	KubeletNotReady    container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error:
	cni plugin not initialized
解决办法：
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo systemctl restart containerd.service
AppArmor是一种Linux内核安全模块，用于应用程序的强制访问控制（MAC）和应用程序沙箱化，类似selinux。
```

10 . 利用java虚拟机，使用.jnlp文件登录服务器，安装低版本jdk(具体查看install jdk7)
```
sudo apt update 
sudo apt install default-jdk 
sudo apt install icedtea-netx icedtea-plugin
javaws file.jnlp
# 出现 duplicate exists
	ps -aux | grep javaws
	kill -9 pid
# 出现network connection has been dropped
	返回web ui，并reset idrac，重新下载jnlp文件
```

11 . 服务器禁止了密码登录，使用密钥文件登录
  eg: local--10.13.3.106, target--10.13.3.107
  local: 需要具有本机的公钥、私钥文件
  target settup:
```
vim /etc/ssh/sshd_config
	PubkeyAuthentication  yes                                  # 使用密钥文件连接
	AuthorizedKeysFile      .ssh/authorized_keys     # 指定公钥存放路径
	PasswordAuthentication  no                                # 禁止密码登录
vim ~/.ssh/authorized_keys
	粘贴 local machine 公钥内容
systemctl restart sshd
rsync -avz /etc/ldap/ldap01_slapd_key.pem 10.13.3.107:/root/  -e "ssh -i ~/.ssh/id_rsa"     # 测试
```
