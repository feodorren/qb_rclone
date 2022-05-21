# qb_rclone
本文只是记录了一次搭建qBittorrent并且通过打标签形式，经由rclone上传至GoogleDrive（OneDrive同理）的测试。

# 特别感谢

https://blog.ba7lcy.com/windows-xia-shi-yong-sh-jiao-ben-yi-qbittorrent-rclone-zi-dong-shang-chuan-jiao-ben-wei-li/

https://hostloc.com/thread-639215-1-1.html


## 其它参考：

https://hostloc.com/thread-612238-1-1.html
https://p3terx.com/archives/offline-download-of-onedrive-gdrive.html


以及感谢其它未提及人员对我的帮助！


# 0. 准备工作、搭建环境及可能用到的工具
putty, bt.cn

# 1. 安装

## 1.1 安装jq

debian & linux

  ```
  apt-get install -y jq
  ```

centos 下

  ```
  yum install -y jq
  ```


## 1.2 安装qbittorrent

https://github.com/userdocs/qbittorrent-nox-static/releases 选择一下适合自己电脑的版本。用ssh登录到debian，执行以下命令

下载安装。我的电脑是x86的，所以选择x86_64-qbittorrent-nox：
```
wget -qO /usr/local/bin/x86_64-qbittorrent-nox https://github.com/userdocs/qbittorrent-nox-static/releases/download/release-4.4.2_v2.0.6/x86_64-qbittorrent-nox
chmod 700 /usr/local/bin/x86_64-qbittorrent-nox
```

把二制文件放在bin目录下面，给二进制文件700权限

给qBittorrent编写进程守护文件，让其开机启动，也可以手动让程序停止或开始。

输入，然后回车，如果没有nano命令，用vi也可以。注意这里用了简写qbt的名字，原名字太长，影响输入：

```
nano /etc/systemd/system/qbt.service
```

然后添加如下内容：
```[Unit]
Description=qBittorrent Service
After=network.target nss-lookup.target

[Service]
UMask=000
ExecStart=/usr/local/bin/x86_64-qbittorrent-nox --profile=/usr/local/etc

[Install]

WantedBy=multi-user.target
```
Ctrl+O 保存，Enter保存文件名，Ctrl+X退出nano编辑器。

请注意：–profile=/usr/local/etc这个选项，表示qBittorrent的配置目录，如果不写，就是在当前用户的主目录，例如root目录下面。我建议按上面的选项填写，这样规范一些。

启动关闭命令：
```
systemctl enable qbt   #开机启动，第一次必须要执行此命令
systemctl start qbt  #启动
systemctl stop qbt  #停止
systemctl status qbt  #软件运行状态查询
```

到目前为止，qBittorrent就算安装成功了，打开浏览器，输入：IP:8080，就可以打开qBittorrent的网页界面了。

用户名：admin
密码：adminadmin

如果是PT玩家，可以自行关闭DHT。

在左侧边栏右键标签，添加以下：
```
  Wait_to_upload,
  Uploading,
  Uploaded,
  Not_upload
```


## 1.3 安装rclone

a. 请自行获取google drive api 下的ID  和 secret

  https://rclone.org/drive/
  参考Making your own client_id章节


b. debian 系统下的命令
  ```
    curl https://rclone.org/install.sh | sudo bash
  ```

  安装完毕后，输入rclone config进行配置
  参考https://rclone.org/drive/ 的Configuration 章节


c. 远程获取token

  下载rclone windows版本到本地

  打开cmd面板

  ```
    cd c:/Users/Administrator/Desktop/reclone文件夹
  ```   

  ```
    rclone authorize "drive" "xxxxxxxxxxxxxxxxxxx此处省略4行"
  ```

  生成token后复制到debian命令行界面 。

  至此reclone配置完毕。


## 1.4 配置自动上传至Google Drive脚本qb_auto_rclone.sh

```
  wget https://raw.githubusercontent.com/feodorren/qb_rclone/main/qb_auto_rclone.sh
```
### 编辑qb_auto_rclone.sh文件 
```
nano qb_auto_rclone.sh
```
修改以下内容

```
  qb_version="4.4.2" # 改：改为你的实际qb的版本号
  qb_username="admin" # 改：该为你的qb登录用户名
  qb_password="adminadmin" # 改：改为你qb登录的密码
  qb_web_url="http://127.0.0.1:8080" # 查：改为qb的登录地址，一般可以不改
  log_dir="/home/qbauto" # 改：改为你日志运行的路径
  rclone_dest="googledrive1:" # 运行rclone config查看name字段即可；格式就是"XX:"
  from_dc_tag="/Upload" # 改：上传后的相对根目录，可为空
  rclone_parallel="32" # rclone上传线程 默认4
```
修改 完毕，ctrl + o 保存, enter 确认名称, ctrl + x 退出。

```
  chmod 777 qb_auto_rclone.sh
```

```
  bash qb_auto_rclone.sh
```


## 1.5 安装bt.cn面板或者使用crond定时即可。

```
  wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
 ```

安装完毕登陆后，定时计划新建 ，每一分钟运行

```
  bash qb_auto_rclone.sh
```

使用crond计划任务，定时2分钟一次执行该sh文件

```
  */2 * * * * bash qb_auto_rclone.sh
```   

# 2. 使用
在qbittorren UI中，将想上传的种子右键标记 Wait_to_upload, 自动命令将会每分钟运行一次将该标记的文件上传至rclone所新建的googledrive1盘里。



# 3. 其它
感谢网上有很多大佬的文章可以参考研究，令人受益匪浅。







