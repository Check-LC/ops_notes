
0. 环境
	centos7 香港 ip, 1G mem，安装有 gitbook
	gitbook workspace pull from github repo，and push to github
	laptop installed vscode， using the extension remote-ssh
	gitbook 利用 github 发布
# 1. 安装node.js
```
wget https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz
	# gitbook 停止维护，安装最新的nodejs或许不能运行。
tar tvf ./node-v12.16.1-linux-x64.tar.xz    # 查看压缩结构
tar xvf ./node-v12.16.1-linux-x64.tar.xz -C /usr/local/
ln -s /usr/local/node-v12.16.1-linux-x64/bin/node /usr/bin/node
ln -s /usr/local/node-v12.16.1-linux-x64/bin/npm /usr/bin/npm
```
# 2. 安装gitbook

```
npm install gitbook-cli -g    # 安装命令行工具
ln -s /usr/local/node-v12.16.1-linux-x64/bin/gitbook /usr/bin/gitbook
gitbook -V
gitbook -h   
```
# 3. 初始化
## 3.1 创建工作目录并初始化 
```
mkdir gitbook
gitbook init
```
## 3.2 编辑SUMMARY.md
```
# Summary

* [Introduction](README.md)
* [前言](readme.md)
* [第一章](part1/README.md)       # [命名](路径----以SUMMARY.md文件为基准的相对路径)
    * [第一节](part1/1.md)
    * [第二节](part1/2.md)
    * [第三节](part1/3.md)
    * [第四节](part1/4.md)
* [第二章](part2/README.md)
* [第三章](part3/README.md)
* [第四章](part4/README.md)
```
中括号里是这个目录(章节)名，小括号里是路径，如果没有，程序将自动创建

## 3.3 编辑后执行
```
gitbook init    可以对本书做初始化，对summary的路径做生成。
gitbook build   会根据项目的目录结构、`README.md` 文件和 `SUMMARY.md` 文件生成一个具有层次结构的网站
gitbook serve   可以发布本书在本地

git pull -->  编辑 --> gitbook init(有需要修改summary时执行) --> gitbook build(第一次需要build) --> git add . --> git commit --> git push
```
# 4. github 新建仓库(建好后Git pull 作为gitbook 工作区)
## 4.1 新建仓库并添加ssh秘钥，便于clone和push
- 1
![|600](repo-create.png)

- 2
![|400](repo-create2.png)

- 3
 ![|500](sshkey.png)

## 4.2 解决Git clone from http，push时出现验证失败
![](Pasted%20image%2020230824231805.png)

```
git remote -v    # 查看当前远程源
git remote rm origin  # 移除
git remote add origin git@github.com:yychuyu/linux-system-programming.git    # 添加ssh方式的clone源
git push --set-upstream origin master    # 直接执行git push是无法推送代码的，需要设置一下上游要跟踪的分支，与此同时会自动执行一次git push命令
```
### 4.2.1 之前由http方式的push，产生的问题
```
git push --set-upstream origin main
	To git@github.com:LONGClong/personalnote.git
	 ! [rejected]        main -> main (fetch first)
	error: failed to push some refs to 'git@github.com:LONGClong/personalnote.git'
	hint: Updates were rejected because the remote contains work that you do
	hint: not have locally. This is usually caused by another repository pushing
	hint: to the same ref. You may want to first merge the remote changes (e.g.,
	hint: 'git pull') before pushing again.
	hint: See the 'Note about fast-forwards' in 'git push --help' for details.
解决：
	执行 `git pull` 命令：这将从远程仓库拉取最新的提交历史记录并尝试合并到您的本地分支中
	随便新增一个文件
	git add .  && git commit -m "test1" && git push -set-upstream origin main
```
# 5 gitbook 发布到 github pages
- 点击进入仓库，点击仓库上方的设置，进入设置后点击左边侧栏 pages
- 设置source：deploy from a branch；设置branch：main 分支以及目录。
- 保存后查看仓库上方 action，等待成功后，查看设置已经有分配的 url。
# 6. vscode 连接远程机器开发
（真牛，远程主机只有 500M mem居然可以使用，idea 远程开发需要1G以上Mem）
## 6.0 在 win 安装vscode，并解决不能实现 ssh 的问题
- 归根结底原因是，remote-ssh 拓展安装的配置文件修改了主机的ssh配置文件的权限，二者有冲突，不能单纯删除某个
-  vscode安装好powershell 环境
-  GitHub下载这个项目: git clone https://github.com/PowerShell/openssh-portable.git
-  管理员身份打开 vscode，进入项目的contrib\win32\openssh 目录，此命令以管理员身份运行 powershell :  Start-Process powershell -Verb runAs
- 执行脚本:  .\\FixUserFilePermissions.ps1 -Confirm:$false
- 如果报错 " 无法加载文件，系统禁止运行脚本 "，则运行此命令: Set-ExecutionPolicy RemoteSigned， 选择 Y
- 再次运行脚本： .\\FixUserFilePermissions.ps1 -Confirm:$false
- 恢复上一个配置： Set-ExecutionPolicy RemoteSigned， 选择 N
- 进行vscode remote-ssh
## 6.1 安装 remote ssh，在 extension 中搜索
![|600](attachments/Pasted%20image%2020230825095711.png)
## 6.2 配置，点击 + 号添加远程对象，选择当前本地用户下的 ssh 配置
![|600](attachments/Pasted%20image%2020230825095759.png)

## 6.3 输入密码后登录进入 welcome 页面，选择 open folder，即可编写。

# 7. 使用流程：
1. github 创建仓库
2. clone 仓库
3. 在克隆的仓库的目录下使用 gitbook init ，将生成一些必要文件
4. 在 SUMMARY.md 编辑结构，即章节名和章节各自的路径
5. 在工作目录下创建相关的章节目录和文件
6. gitbook build ---- 将在_book目录下创建相应的 html 文件
7. git add commit push 到 github 仓库
8. gitbook 将同步 github 内容