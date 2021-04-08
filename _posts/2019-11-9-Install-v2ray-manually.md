---
layout: post
title: "v2ray （ws+tls）手动安装过程记录"
subtitle: ''
date: 2019-11-09 23:30:00
author: "突击核"
header-style: text
hidden: false
tags:
  - nginx
  - v2ray
  - ubuntu
  - 科学上网
---
#v2ray （ws+tls）手动安装过程记录

##环境
| OS  | Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64) |
|-----|-------------------------------------------------------|
| CPU | 1core @ 2.7GHz                                        |
| RAM | 1GB                                                   |
| SSD | 15GB RAID-10                                          |
| 架构  | KVM                                                   

##安装v2ray与Nginx
使用官方推荐的方式安装v2ray：`bash <(curl -L -s https://install.direct/go.sh) `

使用lnmp一键安装包安装lnmp环境（使用Nginx是因为同一台服务器上需要使用Nginx提供的其他服务，如果仅仅是需要实现ws+tls伪装加密，建议使用Caddy，更加小巧且会自动申请证书并自动更新，更加方便）。

##配置Nginx
假设使用`tokyo.39hope.com`作为连接的域名。执行`lnmp vhost add`创建初始配置，添加SSL时选择是会自动生成配置好SSL的配置文件。然后编辑配置文件：`/usr/local/nginx/conf/vhost/tokyo.39hope.com.conf`  

（可选）删除80端口的监听，然后删除443端口下全部location的内容，改为一下内容：
```
location / {  
	           #向后端传递访客IP  
	           proxy_set_header X-Real-IP $remote_addr;  
	           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
	           #设定需要反代的域名，可以加端口号  
	           proxy_pass https://github.com/;  
	           #替换网站内容  
	           sub_filter 'https://github.com/' 'tokyo.39hope.com';  
	           # websocket设定，V2ray使用，这里的设置要和v2ray的设置一致，对于 V2Ray，/ray 和 /ray/ 是不一样的  
	           location /astral {  
	               proxy_redirect off;  
	               #proxy_pass http://127.0.0.1:10000;  
	               proxy_http_version 1.1;  
	               proxy_set_header Upgrade $http_upgrade;  
	               proxy_set_header Connection "upgrade";  
	               proxy_set_header Host $http_host;  
	               proxy_intercept_errors on;  
	               if ($http_upgrade = "websocket" ){  
	                   proxy_pass http://127.0.0.1:10000;  
	               }  
	           }  
	       }  
```
重启Nginx服务：`/usr/local/nginx/sbin/nginx -s reload`  ，然后尝试访问[https://tokyo.39hope.com](https://tokyo.39hope.com) 成功显示GitHub界面即为设置成功。
## 配置v2ray
最后修改v2ray配置：`/etc/v2ray/config.json`

```{  
    "log": {  
        "loglevel": "warning",  
        "access": "/var/log/v2ray/access.log",  
        "error": "/var/log/v2ray/error.log"  
    },  
    "inbounds": [  
        {  
            "port": 10000,  
            "listen": "127.0.0.1",  
            "protocol": "vmess",  
            "settings": {  
                "clients": [  
                    {  
                        "id": "b831381d-6324-4d53ad4f-8cda48b30811",
                        "alterId": 64  
                    }  
                ]  
            },  
            "streamSettings": {  
                "network":"ws",  
                "wsSettings": {  
                    "path": "/astral"  
                }  
            }  
        }  
    ],  
    "outbounds": [  
        {  
            "protocol": "freedom",  
            "settings": {}  
        }  
    ]  
}  
```

完成后使用`service v2ray start`启动服务，配置完成

