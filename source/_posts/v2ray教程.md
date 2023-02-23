---
title: v2ray教程
categories:
  - - v2ray
  - - Linux
tags:
  - vps
  - 服务器
date: 2023-02-23 17:15:00
---
# 原因
+ 现在科学上网稍微有点难度了，shadowsocks频繁的被墙，还好有V2Ray，使用更加可靠的V2Ray搭建自己的利器

<!--more-->

***
# 配置服务器
环境：服务器系统centos 7 64位
## 安装脚本
>curl -Ls https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh | sudo bash
## 获取用户ID
之前的脚本，不用手动配置脚本即可使用，现在使用以上脚本，需要自己配置config.json文件，首先获取用户ID，运用指令：
>cat /proc/sys/kernel/random/uuid 
创建一个用户 id ，并记住这个id号；
``` 
[root@xxxx ~]# cat /proc/sys/kernel/random/uuid 
08ef6234-dcc0-45d1-9954-f9490cb2beb2
```
## 配置
配置文件路径为/usr/local/etc/v2ray/config.json，可以使用“vi”指令创建并打开文本，具体指令如下：
>vi /usr/local/etc/v2ray/config.json
``` 
{
  "inbounds": [{
    "port": 12345,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "6476363c-5e3d-4aa3-b2d6-18e593648wet",
          "level": 1,
          "alterId": 0
        }
      ]
    }
  }],
  "log": {
        "access": "/var/log/v2ray/access.log",
        "error": "/var/log/v2ray/error.log",
        "loglevel": "info"
    },
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
``` 
直接复制配置即可使用，id就是上面第二步获取的用户id，也可以是自己生成的UUID
## 启动V2Ray
在首次安装完成之后，V2Ray不会自动启动，需要手动运行上述启动命令。而在已经运行V2Ray的VPS上再次执行安装脚本，安装脚本会自动停止V2Ray 进程，升级V2Ray程序，然后自动运行V2Ray。在升级过程中，配置文件不会被修改。
+ systemctl start v2ray 启动
+ systemctl stop v2ray 停止
+ systemctl restart v2ray 重启
关于软件更新：更新 V2Ray 的方法是再次执行安装脚本！再次执行安装脚本！再次执行安装脚本！

# 打开防火墙
如果centos系统防火墙是开启的，上面第三步你用了一个端口，因此你需要打开这个端口，指令如下
添加开放端口
>firewall-cmd --zone=public --add-port=12345/tcp --permanent

重载防火墙配置，不然查看开放端口都查不到，也不能用，重载配置后即可
>firewall-cmd --reload

如果哪一天发现怎么无法使用了，有可能是IP被屏蔽，也有肯能是端口被封，这个时候就需要换个端口，别忘记防火墙开启新端口，那旧端口就可以删除了：
删除端口：
>firewall-cmd --zone=public --remove-port=12345/tcp --permanent

# 其它
自 2022 年 1 月 1 日起，服务器端将默认禁用对于 MD5 认证信息的兼容。任何使用 MD5 认证信息的客户端将无法连接到禁用 VMess MD5 认证信息的服务器端。
VMess 协议的 MD5 认证信息将淘汰，VMess AEAD 协议已经经过同行评议并已经整合了相应的修改，作为对 MD5 的替代。
最好的方法是启用 AEAD ，将服务端和客户端的 alterId 设置为 0 会启动 AEAD。设置好后，即可使用。一些过于古老的客户端没有 alterId 的设置，建议更新客户端，如果实在不方便更新客户端，可以使用下面的方法修改环境变量强制兼容 MD5 。
在服务器端可以通过设置环境变量 v2ray.vmess.aead.forced = true 以关闭对于 MD5 认证信息的兼容。 或者 v2ray.vmess.aead.forced = false 以强制开启对于 MD5 认证信息 认证机制的兼容
V2RAY_VMESS_AEAD_FORCED=false
如果使用 systemctl 启动 v2ray 服务，可以修改 v2ray.service 文件
>vi /etc/systemd/system/v2ray.service 

修改为：
``` 
[Unit]
Description=V2Ray Service
Documentation=https://www.v2fly.org/
After=network.target nss-lookup.target

[Service]
User=nobody
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/bin/env v2ray.vmess.aead.forced=false /usr/local/bin/v2ray -config /usr/local/etc/v2ray/config.json
Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
``` 

执行
systemctl daemon-reload
systemctl restart v2ray

参考：http://loonlog.com/2020/10/5/v2ray-server-new/