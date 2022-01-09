## Linux 系统目录结构 FHS
- /：根目录
- /bin：可执行文件，比如 ls 命令
- /boot：引导程序文件，内核，以及 initrd 等文件
- /dev：设置文件，比如磁盘设备
- /etc：系统范围的配置文件
- /home：用户 home 目录，个人用户的配置
- /media：可移除的媒体，cd-rom 等的挂载点
- /lib 和 /lib64：/bin 和 /sbin 中用到的库文件存放位置
- /mnt：临时挂载点
- /opt：可选的应用包，一般用于存放一些直接提供二进制程序的非开源包
- /proc：虚拟问价系统
- /root：root 用户的 home 目录
- /run：存放一些 pid 和 socket 文件
- /sbin：系统的可执行文件，init ，mount 等
- /sys：非FHS保准，但是大部分发行版都有，虚拟文件系统，用来对内核和设备驱动做设置
- /usr：Unix Software Resource 绝大多数的程序和应用工具安装在这里，结果和/非常相似
- /usr/bin
- /usr/lib
- /usr/share：和计算机
- /usr/src：源代码存放路径，如Linux 内核源码
- /var：在程序运行中内容不断变化的文件，比如日志
- /tmp：临时文件系统，重启后内容丢失

## 操作文件的几个常用命令
Linux 中一切皆文件
- ls：list列出目录内容
- cat：输出文件内容到标准输出
- less：查看文件内容
- more：查看文件内容
- head：查看文件头部
- tail：查看文件尾部
- nano：编辑文件的工具
- grep：查找文本中指定关键词的行

## 常见的服务管理方式
- 常见的服务有
  - SSH 用于能随时连接到服务器，提供这个服务的程序是 sshd
  - cron 提供定时任务的服务，提供这个服务的程序是 crond
- systemctl status crond：查看某个服务的状态
- systemctl start crond：启动某个服务
- systemctl stop crond：停止某个服务
- systemctl enable crond：设置某个服务开机启动
- systemctl disable crond：移除某个服务开机启动
- systemctl restart crond：重启某个服务

## 日志与日志查询方式
- journalctl -x：查看日志
- journalctl -xe：跳到尾部查看日志
- 通过直接查看文本的方式查询
  - /var/log/message：全局系统日志，包括登录，对服务启停认证等
  - /var/log/lastlog：不是一个文本文件，需要 lastlog 命令读，保存了最近的用户登录信息
  - /var/log/yum.log：最近通过yum 安装的程序的日志
  - /var/log/cron：定时任务日志
  - /var/log/boot.log：启动日志
  - /var/log/kern：内核日志，也可以通过 dmesg 查看