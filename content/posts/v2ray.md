---
title: "V2Ray and Related"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "v2ray",
    "bbr",
    "cdn",
]

comment: true
toc: true
auto_collapse_toc: true
---
# V2Ray and Others

- [ ] FQ V2Ray
- [ ] 加速 BBR
- [ ] 伪装
    - [ ] 免费域名
    - [ ] websocket+tls+web
    - [ ] CDN

## FQ V2Ray

```console
# # Run as root on VPS

# bash <(curl -L -s https://install.direct/go.sh)

# vim /etc/v2ray/config.json

{
  "inbounds": [{
    "port": 23581,  >>> will be used in client
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "ceb793e6-49cf-25d8-e4de-ae542e62748e",  >>> will be used in client
          "level": 1, 
          "alterId": 64  >>> change to 32, will be used in client
        }
      ]
    }
  }],
... ...

# systemctl enable v2ray
# systemctl start v2ray
```

References:
- https://github.com/hijkpw/scripts/blob/master/centos_install_v2ray.sh
- https://tlanyan.me/v2ray-tutorial/

## 加速 BBR

https://umrhe.com/a-key-to-install-the-latest-kernel-and-open-the-bbr-acceleration-script.html

```console
# wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
# chmod +x bbr.sh
# 此脚本会升级Kernel，如果Kernel版本低于4.9.0。如果是Ubuntu 18.04，不会做升级。
# ./bbr.sh
```


## 免费域名

域名申请 www.freenom.com。

1. 找一个能用的VPN登录freenom，申请`lifeisfun.ml`。
2. 注意下单的时候会显示你当前的IP，可以通过 dp-ip.com 查询到当前IP属于那个地区。貌似如果区域是中国的话，会不成功。
3. 把`Account Details`把地址改成第二步中相应的地区。
4. 下单，成功了之后在`Services >>> My Domains`可以看到自己的域名。
5. 在`Manage Freenom DNS`的列表中加入解析`lifeisfun.ml`和`www.lifeisfun.ml`到VPS IP的Records。这一步主要是为了后面，a)生成SSL certificate和b)v2ray客户端伪装域名。


## websocket+tls+web

此方法可以使得通过 lifeisfun.ml 来访问时就是正常的网络访问，而使用 lifeisfun.ml/great-place-to-live 来访问就是科学上网（通过v2ray客户端来配置）。

而且所有流量都包裹在SSL/TLS的包中，不容易被解析。一个看似对一个nginx server的https包，里面包裹着http ws（vmess）的包，再里面便是真实的数据包。

### 1. 生成证书
```console
# apt install certbot

# # 依赖于上面设置的域名解析，如果 lifeisfun.ml 不能被解析会报错
# # --standalone 意思是将来使用生成的证书来配置nginx
# certbot certonly --standalone -d lifeisfun.ml -d www.lifeisfun.ml
```

### 2. 安装nginx
```console
# apt install nginx

# # 如果文件不存在，则添加以下内容
# cat /etc/nginx/conf.d/default.conf

server {
    listen 80;
    server_name www.lifeisfun.ml lifeisfun.ml;  # >>> customize this line 
    rewrite ^(.*) https://$server_name$1 permanent;
}

server {
    listen       443 ssl http2;
    server_name www.lifeisfun.ml lifeisfun.ml;  # >>> customize this line
    charset utf-8;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    # >>> Customize below two lines using `certbot certificates` to
    # list your cert. 
    ssl_certificate /etc/letsencrypt/live/lifeisfun.ml/fullchain.pem;  
    ssl_certificate_key /etc/letsencrypt/live/lifeisfun.ml/privkey.pem;

    access_log  /var/log/nginx/xxxx.access.log;
    error_log /var/log/nginx/xxx.error.log;

    root /usr/share/nginx/html;
    location / {
        proxy_pass https://github.com/;
    }

    # >>> Access to this url will be redirected to v2ray.
    location /great-place-to-live {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:24817;  # >>> 24817 is v2ray's port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        # Show real IP in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 3. 更新v2ray的配置
```console
# # streamSettings and listen sections are newly added.
# cat /etc/v2ray/config.json
{
  "inbounds": [{
    "port": 24817,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "776bc45d-a118-4913-b3ae-5fd426258c8e",
          "level": 1,
          "alterId": 32
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/great-place-to-live"
        }
      },
    "listen": "127.0.0.1"
  }],
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

### 4. 在路由器上面配置V2ray客户端

注意`服务器端口`，`伪装域名`和`路径`的配置。

![](/forgetful/images/v2ray-router-client.png) 

## CDN

1. 注册登录 www.cloudflare.com。
2. 跟着提示一步步走，选择Free的CDN。
3. 在Freenom上面`Management Tools` >>> `Nameservers`里面添加cloudflare上面给出的两个CDN。
4. 大概要等一会儿生效。
5. 通过`ping www.lifeisfun.ml`和`ping lifeisfun.ml`，来查看是不是使用了CDN。或者通过 www.lifeisfun.ml/cdn-cgi/trace 或者 lifeisfun.ml/cdn-cgi/trace 查看。

