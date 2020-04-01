---
title: 用Shadowsocks加速你的Git
abbrlink: 3aeb306c #38412f22 # 38412f22
date: 2020-03-28 18:02:52
tags: [shadowsocks, proxy, git]
categories: [科学上网]
---

# 使用Shadowsocks加速你的Git

<iframe width="560" height="315" src="https://www.youtube.com/embed/hKBhOXlQc4c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<!-- more -->

## 序言
> 因为平时经常使用git clone项目，而github的服务器位于美国。所以clone的速度非常的慢，慢到惊人的几十kb甚至几kb，clone一个项目要花费很久的时间，而Shadowsocks是一个非常主流的代理软件，可以通过配置Git走 Shadowsocks代理。

## Shadowsocks监听的本地端口号
Shadowsocks在不同平台监听的端口号(port)不一致
  {% tabs shadowsocks-port-tag, 2 %}
  <!-- tab Mac OS X @apple -->
  ``` json
  port: 1086
  ```
  <!-- endtab -->
  <!-- tab Windows @windows -->
  ``` json
  port: 1080
  ```
  <!-- endtab -->
  <!-- tab Linux @linux -->
  ``` json
  port: 1080
  ```
  <!-- endtab -->
  {% endtabs %}

## 配置git代理
Shadowsocks使用的是socks5代理，下面演示了在不同平台配置git代理

  {% tabs git-proxy-tag, 2 %}
  <!-- tab Mac OS X @apple -->
  ``` bash
  git config --global http.proxy 'socks5://127.0.0.1:1086'
  git config --global https.proxy 'socks5://127.0.0.1:1086'
  ```
  <!-- endtab -->
  <!-- tab Windows @windows -->
  ``` bash
  git config --global http.proxy 'socks5://127.0.0.1:1080'
  git config --global https.proxy 'socks5://127.0.0.1:1080'
  ```
  <!-- endtab -->
  <!-- tab Linux @linux -->
  ``` bash
  git config --global http.proxy 'socks5://127.0.0.1:1080'
  git config --global https.proxy 'socks5://127.0.0.1:1080'
  ```
  <!-- endtab -->
  {% endtabs %}

## 取消代理
如果不想让git再使用代理，可以执行下面的命令取消。
``` bash
git config --global --unset https.proxy
git config --global --unset http.proxy
```
