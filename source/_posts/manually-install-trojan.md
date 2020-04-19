---
title: 谷歌云搭建trojan+全平台配置(IOS Mac Android Windows Linux)科学上网
abbrlink: 48a35741
date: 2020-04-04 17:42:21
tags:
---

## 什么是[trojan](https://github.com/trojan-gfw/trojan)
> An unidentifiable mechanism that helps you bypass GFW.

<!-- more -->

## 购买vps
* [免费注册Google Cloud](https://console.cloud.google.com/)
* [点击优惠注册vultr](https://www.vultr.com/?ref=7133201)

## [在线文本编辑器](https://www.editpad.org/)

## 更新系统
  * 进入系统后需要更新系统否则，部分软件无法安装
    {% tabs %}
    <!-- tab Centos @linux -->
    ``` bash
    yum install epel-release -y && yum update -y && yum install curl wget -y
    ```
    <!-- endtab -->
    <!-- tab Debian/Ubuntu @linux -->
    ``` bash
    apt update -y && apt install curl wget -y
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

## 安装SSL免费证书
  * 安装cerbot
    {% tabs certbot-tabs %}
    <!-- tab Centos @linux -->
    ``` bash
    yum install certbot -y
    ```
    <!-- endtab -->
    <!-- tab Debian/Ubuntu @linux -->
    ``` bash
    apt install certbot -y
    ```
    <!-- endtab -->
    {% endtabs %}

  * 生成证书
    把域名和邮箱替换成自己的
    ``` bash
    certbot certonly --standalone --agree-tos -n -d www.duyuanchao.me -d duyuanchao.me -m shellhub.me@gmail.com
    ```
  * 自动更新ssl证书
    ``` bash
    echo "0 0 1 */2 * service nginx stop; certbot renew; service nginx start;" | crontab
    ```

## 使用官方脚本安装trojan
  * 安装cerbot
    {% tabs trojan-tabs %}
    <!-- tab wget安装 -->
    ``` bash
    bash -c "$(wget -O- https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
    ```
    <!-- endtab -->
    <!-- tab curl安装 -->
    ``` bash
    bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
    ```
    <!-- endtab -->
    {% endtabs %}
  * 更新trojan配置文件
    trojan配置文件位于:`/usr/local/etc/trojan/config.json`
    ``` bash
cat <<EOF >/usr/local/etc/trojan/config.json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password1",
        "password2"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/letsencrypt/live/trojan.duyuanchao.me/fullchain.pem",
        "key": "/etc/letsencrypt/live/trojan.duyuanchao.me/private.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "alpn_port_override": {
            "h2": 81
        },
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": "",
        "cafile": ""
    }
}
EOF    
    ```
  * 查看trojan状态
    ``` bash
    systemctl status trojan
    ```
  * 启动trojan
    ``` bash
    systemctl start trojan
    ```
  * 重启trojan
    ``` bash
    systemctl restart trojan
    ```
  * 开机自启
    ``` bash
    systemctl enable trojan
    ```

## 客户端的配置

{% tabs trojan-client %}

<!-- tab IOS @apple -->
[小火箭🚀Shadowrocket](https://apps.apple.com/us/app/shadowrocket/id932747118)
<!-- endtab -->

<!-- tab MAC OS X @apple -->
* [trojan](https://github.com/trojan-gfw/trojan/releases) + 系统设置socks5代理
* [trojan](https://github.com/trojan-gfw/trojan/releases)+[V2rayU](https://github.com/2dust/v2rayN/releases)
<!-- endtab -->

<!-- tab Android @android -->
[trojan官方出品igniter](https://github.com/trojan-gfw/igniter/releases)
<!-- endtab -->

<!-- tab Windows @windows -->
[trojan](https://github.com/trojan-gfw/trojan/releases)+[V2rayN](https://github.com/2dust/v2rayN/releases)
<!-- endtab -->

<!-- tab Linux @linux -->
[trojan](https://github.com/trojan-gfw/trojan/releases)+[Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en)
<!-- endtab -->

{% endtabs %}
