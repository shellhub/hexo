---
title: v2ray ws tls nginx
abbrlink: 9311723b
date: 2020-03-22 21:54:10
categories: [科学上网]
---
# V2ray+WebSocket+TLS+Nginx 伪装流量科学上网(干货)

<iframe width="560" height="315" src="https://www.youtube.com/embed/FnC1_pJVflY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<!-- more -->

## 简介
> 本文介绍V2ray+WebSocket+TLS+Nginx的方式实现更强的流量伪装。通过这种方式减少vps服务器被墙的概率。

## 本教程所需要的工具
你可以可以直接通过其他纯文本编辑器编辑文件，如Notepad Atom VScode 甚至Vim
* [在线文本编辑器](https://www.editpad.org/)

## 购买vps
* [点击优惠注册vultr](https://www.vultr.com/?ref=7133201)
* [免费注册Google Cloud](https://console.cloud.google.com/)

## 进入管理员(root)
* 需要进入root才能安装各种各样的服务
  ``` bash
  sudo su
  ```

## 更新系统
  * 进入系统后需要更新系统否则，部分软件无法安装
    {% tabs %}
    <!-- tab Centos -->
    ``` bash
    yum install epel-release -y
    yum update -y
    ```
    <!-- endtab -->
    <!-- tab Debian/Ubuntu -->
    ``` bash
    apt update -y
    ```
    <!-- endtab -->
    {% endtabs %}

## 开启Google BBR加速(可选)
Linux内核4.9以及上默认支持了该算法，直接启用就行
  * 检查Linux内核版本
    ``` bash
    uname -r
    ```
  * 添加并启用Google BBR模块到内核
    ``` bash
    echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
    echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
    sysctl -p
    ```
  * 检查是否成功加入Google BBR模块到内核中
    ``` bash
    sysctl net.ipv4.tcp_available_congestion_control
    ```
  * 验证是否启用了Google BBR
    ``` bash
    lsmod | grep bbr
    ```
  > 如果安装失败，推荐使用一键安装脚本。[视频教程](https://www.youtube.com/watch?v=5E9bRJ_oCUg&t=1s)

## 安装并配置v2ray
  * 官方脚本安装v2ray
    ``` bash
    bash <(curl -L -s https://install.direct/go.sh)
    ```
  * 获取uuid: https://www.uuidgenerator.net/
  * 获取随机字符串: https://www.random.org/strings/
  * 修改v2ray配置文件
    v2ray配置文件保存在`/etc/v2ray/config.json`,替换下方的端口号`port`, 密码`id`, 及路径`path`字段值。如果实在太懒直接用我的也行(不推荐)。
``` bash
cat <<EOT > /etc/v2ray/config.json
{
  "inbounds": [
    {
      "port": 9876,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray"
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
EOT
```
  * 检查配置文件并重启v2ray服务
    ``` bash
    /usr/bin/v2ray/v2ray -test -config=/etc/v2ray/config.json
    systemctl restart v2ray
    ```
## 安装nginx服务器
  * 安装nginx服务器
    ``` bash
    yum install nginx -y
    ```
  * 查看nginx状态
    ``` bash
    systemctl status nginx
    ```
  * 启动nginx服务器
    ``` bash
    systemctl start nginx
    ```
## 安装SSL免费证书
  * 安装cerbot
    ``` bash
    yum install certbot -y
    ```
  * 生成证书
    把域名和邮箱替换成自己的
    ``` bash
    certbot certonly --standalone --agree-tos -n -d www.duyuanchao.me -d duyuanchao.me -m shellhub.me@gmail.com
    ```
  * 自动更新ssl证书
    ``` bash
    echo "0 0 1 */2 * service nginx stop; certbot renew; service nginx start;" | crontab
    ```

## 创建nginx配置文件
更改`ssl_certificate` `ssl_certificate_key`` server_name` `proxy_pass` `location`
等字段值
* 创建nginx配置文件:`/etc/nginx/conf.d/default.conf`
``` bash
cat <<EOT > /etc/nginx/conf.d/default.conf
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  # config ssl
  ssl_certificate       /etc/letsencrypt/live/www.duyuanchao.me/fullchain.pem;
  ssl_certificate_key   /etc/letsencrypt/live/www.duyuanchao.me/privkey.pem;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;
  ssl_session_tickets off;

  ssl_protocols         TLSv1.2 TLSv1.3;
  ssl_ciphers           ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  server_name           duyuanchao.me; # config server_name
    location /ray { # config path
      if (\$http_upgrade != "websocket") {
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:9876; # config proxy_pass
      proxy_http_version 1.1;
      proxy_set_header Upgrade \$http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host \$host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP \$remote_addr;
      proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
EOT
```
* 检查配置文件是否正确  
  ``` bash
  nginx -t
  ```
* 重启nginx服务器
  ``` bash
  systemctl restart nginx
  ```
* 部分Linux需要禁用SELinux
  ``` bash
  setsebool -P httpd_can_network_connect 1 && setenforce 0
  ```

## 下载网页模版并部署到nginx目录
下载一个网页项目部署到nginx服务器上，减少被墙的可能
推荐网站模版
  * https://www.free-css.com/free-css-templates
  * https://colorlib.com/wp/templates/
  ``` bash
  cd /usr/share/nginx/html/
  yum install wget unzip -y
  wget xxxx.zip
  unzip website.zip
  mv website/* .
  ```
  * 访问web服务器
    https://v2ray.duyuanchao.me:443

## 开启https端口号(443)
这里强力建议不要直接把防火墙关闭暴露所有端口号，而是打开防火墙减少服务器被墙的风险。
  * 安装ufw
    > UFW 全称为 Uncomplicated Firewall，是 Ubuntu 系统上默认的防火墙组件, 为了轻量化配置 iptables 而开发的一款工具。UFW 提供一个非常友好的界面用于创建基于IPV4，IPV6的防火墙规则。

    ``` bash
    yum install ufw -y
    ```
  * 查看防火墙状态
    ``` bash
    ufw status
    ```
  * 开启防火墙
    ``` bash
    ufw enable
    ```
  * 关闭防火墙
    ``` bash
    ufw disable
    ```
  * 开启https端口(443)
    ``` bash
    ufw allow 443/tcp
    ```
## v2ray客服端下载
  * [Shadowrocket IOS](https://apps.apple.com/us/app/shadowrocket/id932747118)
  * [V2rayU Mac](https://github.com/yanue/V2rayU/releases)
  * [V2rayN Windows](https://github.com/2dust/v2rayN/releases)
  * [v2rayNG Android](https://github.com/2dust/v2rayNG/releases)
