---
title: rsync文件同步
categories:
  - - Linux
  - - 教程
tags:
  - 服务器
  - centos
date: 2024-04-09 16:36:34
---
# rsync介绍
rsync是Liunx下的远程数据同步工具，可快速同步多台服务器间的文件和目录，并可实现增量同步以减少数据的传输。
rsync有两种常用的认证方式，一种是rsync-daemon方式，另外一种是ssh方式。
daemon 方式与 ssh 方式相比有以下几点不同：
- 1、不需要依赖远程服务器的 sshd 服务，但需要远程服务器开启 rsyncd 服务，本地 rsyncd 服务可不必开启。
- 2、不直接使用远程服务器的真实系统账号，而是虚拟账号和虚拟密码，且可实现无需手动输入密码，同时配置模块对远程同步的目录进行限制。
- 3、对比 ssh 方式，daemon方式安全性更高。

# 安装rsync
CentOS 7.x及以上的版本默认已安装rsync，可以通过命令查看是否安装成功
``` shell
rpm -qa |grep rsync
# rsync-3.1.2-12.el7_9.x86_64 表示已安装
# 如未安装可通过以下命令进行安装
yum -y install rsync
```
# 服务器准备
* 远程服务器192.168.1.1，提供服务，需开启并配置rsyncd
* 本地服务器192.168.1.2，无需开启配置rsyncd
<!--more-->

***

# 远程服务器配置
## 1、配置rsyncd.conf
``` shell
vi /etc/rsyncd.conf
```
## 2、输入以下内容，部分内容可根据情况进行调整
``` bash
# 以 rsync 用户启动进程
uid = rsync
gid = rsync

# 无需让rsync以root身份运行，允许接收文件的完整属性
fake super = yes

# 禁锢推送的数据至某个目录，不允许跳出该目录
# 允许chroot，提升安全性，客户端连接模块，首先chroot到模块path参数指定的目录下
# chroot为yes时必须使用root权限，且不能备份path路径外的链接文件
use chroot = no

# 最大连接数
max connections = 200

# 超时时间
timeout = 300

# pid文件路径
pid file = /var/run/rsyncd.pid

# 锁文件路径
lock file = /var/run/rsync.lock

# 剔除某些文件或目录不同步
exclude = lost+found/

# 记录传输文件日志
transfer logging = yes

# 指定日志文件
log file = /var/log/rsyncd.log

# 日志文件格式
log format = %t %a %m %f %b

# 忽略错误信息
ignore errors

# 对备份数据可读写
read only = false

# 不允许查看模块信息
list = false

# 定义虚拟用户，作为连接认证用户
auth users = rsync_backup

# 定义rsync服务用户连接认证密码文件路径
secrets file = /etc/rsync.password

# 设置不需要压缩的文件
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

# 定义模块信息
[backup]

# 模块注释信息
comment = "backup dir"

# 定义备份数据目录，此处根据实际调整
path = /backup/test/
```
## 3、创建rsync用户
```
id rsync
# 可能返回 id: rsync: No such user

useradd -s /sbin/nologin -M rsync
#（-s创建用户shell，/sbin/nologin表示不登录，—M rsync表示不创建用户rsync家目录）
```
## 4、创建数据备份储存目录并修改属性
```
# 创建的目录 /backup/test/ 要于第2步配置文件中rsyncd.conf的path相同

mkdir /backup/test/
chown -R rsync:rsync /backup/
```
<font color="#F56C6C">必须要从根路径赋值权限，不然在有子目录传输的时候会报权限错误</font>
``` shell
2024/04/09 15:34:56 [1893] params.c:Parameter() - Ignoring badly formed line in config file: ignore errors
2024/04/09 15:34:56 [1893] name lookup failed for x.x.x.x: Name or service not known
2024/04/09 15:34:56 [1893] connect from UNKNOWN (x.x.x.x)
2024/04/09 15:34:56 [1893] rsync on backup/x/x.jpg from rsync_backup@UNKNOWN (x)
2024/04/09 15:34:56 [1893] building file list
2024/04/09 15:34:56 [1893] rsync: change_dir "/x" (in backup) failed: Permission denied (13)
```
## 5、创建认证用户密码文件
``` shell
# 命令中用户rsync_backup必须与/etc/rsync.password里定义虚拟用户名一致
# 命令中test123为认证用户的密码，根据实际进行调整

echo "rsync_backup:test123" >> /etc/rsync.password

chmod 600 /etc/rsync.password
```
## 6、启动rsync服务
``` shell
rsync --daemon
# 如需关闭，输入命令 pkill rsync，则服务停止
```
## 7、检查服务是否正常运行
```
ps -ef |grep rsync
netstat -antlp |grep rsync
# 如netstat未安装，可通过命令进行安装 yum install net-tools
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN      1911/rsync          
tcp        0 128001 10.0.144.2:873          74.48.74.39:47446       ESTABLISHED 1948/rsync          
tcp6       0      0 :::873                  :::*                    LISTEN      1911/rsync     
```
# 本地服务器配置
## 创建认证文件
```
# 命令中test123为远程服务器认证用户的密码，需保持一致
echo "test123" >> /etc/rsync.password
chmod 600 /etc/rsync.password
```
## 本地服务器同步至远程服务器（交互式）
交互式每次提交时需要手动输入认证用户的密码，本例中为test123
```
rsync -aRvuz /home/ rsync_backup@192.168.1.1::backup
```
## 本地服务器同步至远程服务器（免交互式）
免交互式因已配置/etc/rsync.password文件，每次提交时无需输入密码
```
rsync -aRvuz /home/ rsync_backup@192.168.1.1::backup --password-file=/etc/rsync.password
```
## 远程服务器同步至本地服务器
```
# 交互式需手动输入认证用户密码，本例中为test123
rsync -aRvuz rsync_backup@192.168.1.1::backup /home/

# 免交互式无需输入密码
rsync -aRvuz rsync_backup@192.168.1.1::backup /home/ --password-file=/etc/rsync.password
```
注意：
>源目录加了斜线，效果就是将该目录下的内容传输到目标目录下，如/test/表示将目录test下(不含test目录本身)的文件及目录同步至目标目录
源目录不加斜线，效果就是将该目录传输到目标目录下，如/test表示将目录test(含test目录本身)的文件及目录同步至目标目录
>目标目录如果不存在，会自动创建目标目录

# 参数
rsync的命令格式可以为以下六种： 
```
rsync [OPTION]... SRC DEST 
rsync [OPTION]... SRC [USER@]HOST:DEST 
rsync [OPTION]... [USER@]HOST:SRC DEST 
rsync [OPTION]... [USER@]HOST::SRC DEST 
rsync [OPTION]... SRC [USER@]HOST::DEST 
rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST] 
```
对应于以上六种命令格式，rsync有六种不同的工作模式： 
- 1)拷贝本地文件。当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。如：rsync -a /data /backup 
- 2)使用一个远程shell程序(如rsh、ssh)来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。如：rsync -avz *.c foo:src 
- 3)使用一个远程shell程序(如rsh、ssh)来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。如：rsync -avz foo:src/bar /data 
- 4)从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。如：rsync -av root@172.16.78.192::www /databack 
- 5)从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。如：rsync -av /databack root@172.16.78.192::www 
- 6)列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。如：rsync -v rsync://172.16.78.192/www 
由此可知，rsync可以像cp一样从本地拷贝文件备份，可以像scp一样远程拷贝，也可以以rsync服务的方式拷贝文件（隐藏了本地具体目录名称，增强了安全性），关于rsync服务的搭建，后面有专门讲解。注意rsync不能实现直接控制远程机器和远程机器之间的拷贝。

常用参数
```
-v, --verbose详细模式输出
-a, --archive归档模式，表示以递归方式传输文件，并保持所有文件属性不变
-u, --update 仅仅进行更新，也就是跳过已经存在的目标位置，并且文件时间要晚于要备份的文件，不覆盖新的文件
-z,--compress对备份的文件在传输时进行压缩处理
--delete，删除那些目标目录中存在而在源目录中没有的文件
--exclude=PATTERN，指定排除不需要传输的文件模式
```
全部参数
```
-v, --verbose 详细模式输出
-q, --quiet 精简输出模式
-c, --checksum 打开校验开关，强制对文件传输进行校验
-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
-r, --recursive 对子目录以递归模式处理
-R, --relative 使用相对路径信息
# rsync foo/bar/foo.c remote:/tmp/      ## Rsync 参数在/tmp目录下创建foo.c文件，而如果使用-R参数：
# rsync -R foo/bar/foo.c remote:/tmp/   ## Rsync 参数会创建文件/tmp/foo/bar/foo.c，也就是会保持完全路径信息。
-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir 将备份文件(如~filename)存放在在目录下。
-suffix=SUFFIX 定义备份文件前缀
-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
-l, --links 保留软链结
-L, --copy-links 想对待常规文件一样处理软链结
--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结
--safe-links 忽略指向SRC路径目录树以外的链结
-k, --copy-dirlinks  transform symlink to a dir into referent dir
-K, --keep-dirlinks  treat symlinked dir on receiver as dir
-H, --hard-links 保留硬链结
-p, --perms 保持文件权限
-o, --owner 保持文件属主信息
-g, --group 保持文件属组信息
-D, --devices 保持设备文件信息
-t, --times 保持文件时间信息
-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间
-n, --dry-run现实哪些文件将被传输
-W, --whole-file 拷贝文件，不进行增量检测
-x, --one-file-system 不要跨越文件系统边界
-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节
-e, --rsh=COMMAND 指定替代rsh的shell程序
--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息
-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件
--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
--delete 删除那些DST中SRC没有的文件
--delete-excluded 同样删除接收端那些被该选项指定排除的文件
--delete-after 传输结束以后再删除
--ignore-errors 及时出现IO错误也进行删除
--max-delete=NUM 最多删除NUM个文件
--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输
--force 强制删除目录，即使不为空
--numeric-ids 不将数字的用户和组ID匹配为用户名和组名
--timeout=TIME IP超时时间，单位为秒
-I, --ignore-times 不跳过那些有同样的时间和长度的文件
--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0
-T --temp-dir=DIR 在DIR中创建临时文件
--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份
-P 等同于 --partial
--progress 显示备份过程
-z, --compress 对备份的文件在传输时进行压缩处理
--exclude=PATTERN 指定排除不需要传输的文件模式
--include=PATTERN 指定不排除而需要传输的文件模式
--exclude-from=FILE 排除FILE中指定模式的文件
--include-from=FILE 不排除FILE指定模式匹配的文件
--version 打印版本信息
--address 绑定到特定的地址
--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件
--port=PORT 指定其他的rsync服务端口
--blocking-io 对远程shell使用阻塞IO
-stats 给出某些文件的传输状态
--progress 在传输时现实传输过程
--log-format=FORMAT 指定日志文件格式
--password-file=FILE 从FILE中得到密码
--bwlimit=KBPS 限制I/O带宽，KBytes per second
-h, --help 显示帮助信息
```

