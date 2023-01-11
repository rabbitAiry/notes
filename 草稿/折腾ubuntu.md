### 安装
- 使用balenaEtcher烧写U盘（会清空U盘），然后在需要安装的电脑中，从bios选择启动该U盘
- 选择Try or Install Ubuntu（我使用的是22.04版本，可能选项不一样）
- 安装选项
	- 安装类型
		- 选择“与Windows Boot Manager”共存时，会使用硬盘未分区部分进行安装
		- 选择“其他选项”以进行自定义，将用于安装系统的分区挂载为`/`即可

### 避免密码验证（似乎无效）
- 输入`sudo visudo`
- 在`root ALL=(ALL) ALL`下一行输入`你的帐号 ALL=(ALL) NOPASSWD:ALL`

### 双系统导致时间不一致
- 在ubuntu终端中输入`timedatectl set-local-rtc 1`
- 原因见[此博客](https://blog.csdn.net/syluxhch/article/details/128170576) 

### 安装deb文件
- Ubuntu支持直接安装deb文件
- 右键以“软件安装”的方式打开，确认即可

### 安装tar.gz文件
- 解压，文件夹中有应用开发团队提供的安装说明
- 应该将文件解压到`/opt`目录下
- 在“应用程序”中添加软件对应的图标
```sh
# 以idea为例
cd /usr/share/applications
gedit idea.desktop
```
- desktop文件中输入以下内容（以idea为例）
```
[Desktop Entry]
Type=Application
Name=IDEA
Comment=Run IDEA
Icon=【对应软件目录】/【软件图标文件】.png
Exec=【对应软件目录】/idea.sh
Terminal=false
Path=
StartupNotify=false
```

### 安装clash
###### 方法1：使用clash
- [在此](https://github.com/Dreamacro/clash/releases)找到对应版本（github）的clash文件
- 解压文件（gunzip）
- 下载配置文件
- 启动clash
```sh
wget -O config.yaml 订阅链接
wget -O Country.mmdb https://www.sub-speeder.com/client-download/Country.mmdb
./clash -d .
```
- 在系统设置中选择网络>网络代理>手动
	- HTTP和HTTPS代理为`127.0.0.1:7890`
	- Socks代理为`127.0.0.1:7891`
	- 忽略主机默认为`localhost, 127.0.0.0/8,::1`
- 在浏览器输入`clash.razord.top`可访问控制页面
- 配置开机自启动
```sh
su
cd /etc/systemd/system
gedit clash.service
```
- 在clash.service文件中填入以下内容
```
[Unit] 
Description=clash daemon  
[Service] 
Type=simple 
User=root 
ExecStart=【Clash所在目录】/clash -d 【Clash所在目录】/ 
Restart=on-failure  
[Install] 
WantedBy=multi-user.target
```
- 继续设置开机自启动
```sh
sudo systemctl daemon-reload 
sudo systemctl enable clash 
sudo systemctl start clash 
sudo systemctl status clash
```

###### 方法2：使用clash_for_windows
- [在此](https://github.com/Fndroid/clash_for_windows_pkg/releases/tag/0.17.1)找到对应版本的clash文件（我没能成功使用）
