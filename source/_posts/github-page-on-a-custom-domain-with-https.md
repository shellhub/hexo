---
title: Github Pages 添加自定义域名并且开启 HTTPS 支持访问。
tags:
  - Github
  - HTTPS
abbrlink: f14cfd82
date: 2020-04-02 18:33:23
---
# Github Pages 添加自定义域名并且开启HTTPS支持
最近使用Github Page搭建了一个博客网站(基于[hexo](https://github.com/hexojs/hexo) + [next主题](https://github.com/theme-next/hexo-theme-next))并且在Godday用自己的名字注册了一个域名。本期介绍如何给Github Pages 添加自定义域名并且开启 HTTPS 支持访问。
<iframe width="560" height="315" src="https://www.youtube.com/embed/bb1BKHzTJpA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<!-- more -->

## 在Github Pages上发布你的网站
Github提供了一个叫做Github Pages的功能，可以免费把你的静态网页托管在Github。

* 在Github上创建一个名为`github-username.github.io`的仓库，我的Github用户名:[shellhub](https://github.com/shellhub/)，所以对应的仓库名称为: [shellhub.github.io](https://github.com/shellhub/shellhub.github.io)
* 然后发布网站的内容到这个仓库,你可以在本地开发好然后发布到这个仓库


## 添加自定义域名

* 购买域名

  我的域名是从[Goddy](https://www.godaddy.com/)购买的。主要是购买方面支持信用卡，支付宝，微信付款。最重要的是不需要上传身份证。

* 添加一个名为`CNAME`的文件，主要有三种方式创建
  * 第一种方式:在本地的网站根目录创建一个叫做`CNAME`的文件(没有扩展名),然后文件的内容为你的域名，可以是一个一级域名也可以是二级域名。

  * 第二种方式:是直接通过Github项目主页的`Create new file`按钮进行创建。文件名:`CNAME`，文件内容:`一级或二级域名`

  * 第三种方式:直接点击Github项目主页的`Setting`标签,找到`Custom domain`，下方填入您购买的域名，点击`Save`进行保存即可。

## 解析域名
我们需要去Godaddy添加3个`A`记录,`IP`指向分别如下
``` json
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
Github有可能会更新此`IP地址`，可以访问这个链接[获取最新的`IP`](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)

## 强制开启HTTPS
解析完`A`记录后我们需要回到Github `Setting`页面然后找到`Enforce HTTPS`并勾选上，过几分钟后就可以以https+domain的形式去访问Github Pages了。
