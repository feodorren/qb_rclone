# qb_rclone
本文只是记录了一次搭建qBittorrent并且通过打标签形式，经由rclone上传至GoogleDrive（OneDrive同理）的测试。

## 特别感谢

https://blog.ba7lcy.com/windows-xia-shi-yong-sh-jiao-ben-yi-qbittorrent-rclone-zi-dong-shang-chuan-jiao-ben-wei-li/

https://hostloc.com/thread-639215-1-1.html


## 其它参考：

https://hostloc.com/thread-612238-1-1.html
https://p3terx.com/archives/offline-download-of-onedrive-gdrive.html


以及感谢其它未提及人员对我的帮助！


## 准备工作、搭建环境及可能用到的工具。
putty, vs code, bt.cn, docker.

## 安装

## 安装jq

debian & linux

  ```
  apt-get install -y jq
  ```

centos 下

  ```
  yum install -y jq
  ```


## 安装qbittorrent
https://hub.docker.com/r/linuxserver/qbittorrent

特别说明：本人才疏学浅，无法解决某些专业问题。因此指定/downloads目录就在根目录下，防止产生绝对地址引用错误问题。
```shell
  docker run -d \
    --name=qbittorrent \
    -e PUID=1000 \
    -e PGID=1000 \
    -e TZ=Asia/Shanghai \
    -e WEBUI_PORT=8080 \
    -p 8080:8080 \
    -p 6881:6881 \
    -p 6881:6881/udp \
    -v /path/to/appdata/config:/config \
    -v /downloads:/downloads \
    --restart unless-stopped \
    lscr.io/linuxserver/qbittorrent
```
安装完毕后可以在浏览器打开 xxx.xxx.xxx.xxx:8080 访问UI。
如果是PT玩家，可以自行关闭DHT。

在左侧边栏右键标签，添加以下：
```
  Wait_to_upload,
  Uploading,
  Uploaded,
  Not_upload
```


## 安装rclone

a. 请自行获取google drive api 下的ID  和 secret

  https://rclone.org/drive/
  参考Making your own client_id章节


b. debian 系统以下命令
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


## 配置qb_auto_reclone.sh

```
  nano /root/qb_auto_reclone.sh
```

下载库中的qb_auto_reclone.sh，然后用vs code 打开，根据自己的配置修改以下内容后，全选复制整个文档内容。

```shell
  qb_version="4.4.2" # 改：改为你的实际qb的版本号
  qb_username="admin" # 改：该为你的qb登录用户名
  qb_password="adminadmin" # 改：改为你qb登录的密码
  qb_web_url="http://127.0.0.1:8080" # 查：改为qb的登录地址，一般可以不改
  log_dir="/home/qbauto" # 改：改为你日志运行的路径
  rclone_dest="googledrive1:" # 运行rclone config查看name字段即可；格式就是"XX:"
  from_dc_tag="/Upload" # 改：上传后的相对根目录，可为空
  rclone_parallel="32" # rclone上传线程 默认4
```
返回nano界面粘贴完毕，ctrl + o 保存, enter 确认名称, ctrl + x 退出。

```
  chmod 777 qb_auto_rclone.sh
```

```
  bash qb_auto_rclone.sh
```


## 安装bt.cn面板  或者使用crond定时即可。

```
  wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
 ```

安装完毕登陆后，定时计划新建 ，每一分钟运行

```
  bash qb_auto_rclone.sh
```

使用crond计划任务，定时1分钟一次执行该sh文件

```
  */1 * * * * bash qb_auto_rclone.sh
```   

## 使用
在qbittorren UI中，将想上传的种子右键标记 Wait_to_upload, 自动命令将会每分钟运行一次将该标记的文件上传至rclone所新建的googledrive1盘里。



# 其它
感谢网上有很多大佬的文章可以参考研究，令人受益匪浅。







