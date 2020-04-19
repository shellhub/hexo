---
title: 搭建ssrpanel + caddy实现高级飞机场
tags:
  - ssrpanel
  - ssr
  - caddy
categories:
  - 科学上网
abbrlink: 1a073008
date: 2020-04-19 14:31:15
---

## 切换管理员
``` bash
sudo su
```

## 更新系统
{% tabs,update%}
<!-- tab Centos @linux -->
``` bash
yum update -y
yum groupinstall "Development Tools" -y
yum install wget git -y
```
<!-- endtab -->
<!-- tab Debian/Ubuntu @linux -->
``` bash
apt update -y
apt install git build-essential wget -y
```
<!-- endtab -->
{% endtabs %}

## 安装[宝塔面板](https://www.bt.cn/)

{% tabs,bt%}
<!-- tab Centos @linux -->
``` bash
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```
<!-- endtab -->
<!-- tab Debian/Ubuntu @linux -->
``` bash
apt install wget -y && wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```
<!-- endtab -->
{% endtabs %}

> Do you want to install Bt-Panel to the /www directory now?(y/n):
选择`y`开始安装宝塔面板, 安装完成后会显示面板的地址以及面板的用户名及密码，登录后同意协议，即可进入面板页面

* 安装成功后的面板信息
  ``` json
  Complete!
  success
  ==================================================================
  Congratulations! Installed successfully!
  ==================================================================
  Bt-Panel: http://104.197.207.96:8888/d3128078
  username: zhtamcs9
  password: a45d38df
  Warning:
  If you cannot access the panel,
  release the following port (8888|888|80|443|20|21) in the security group
  ==================================================================
  Time consumed: 2 Minute!
  ```

* 查看宝塔面板密码
  ``` bash
  bt default
  ```

## 安装LNMP环境
> 勾选对应的版本就行了，安装完成后需要通过页面安装php扩展`fileinfo`

* Nginx 1.17+
* MYSQL 5.5+ （推荐5.6）
* FPT         (可选)
* PHP 7.1.3+ （必须）
* phpMyadmin  (可选)
* 安装方式: 极速安装

## 添加网站目录
点击 `网站` -> `添加站定` 既可添加站点
* 域名: 如 `ssrpanel.duyuanchao.me`,如果没有直接填写ip地址
* 备注: 随便填写，自动填充: `ssrpanel.duyuanchao.me`
* 网站根目录: 默认填充就行: `/www/wwwroot/ssrpanel.duyuanchao.me`
* FTP: 创建,用户名:`ssrpanel`密码:`ssrpanel`
* 数据库: 不创建
* PHP版本: `PHP-71`默认即可，如果PHP没有安装成功，需要重新安装PHP
点击提交

## 搭建ssrpanel前端
* 切换到站点目录,就是刚才的网站根目录,替换下方的ip为你的域名或者ip
  ``` bash
  cd /www/wwwroot/ssrpanel.duyuanchao.me
  ```
* 克隆[ssrpanel](https://github.com/ssrpanel/SSRPanel.git)源码
  ``` bash
  git clone https://github.com/ssrpanel/SSRPanel.git
  ```
* 切换到SSRPanel目录
  ``` bash
  cd SSRPanel
  ```
* 设置网站目录
  `设置`->`网站目录`->`/www/wwwroot/ssrpanel.duyuanchao.me/SSRPanel`, 保存，运行目录: `/public`
* 伪静态，选择laravel5

  ``` xml
  location / {  
  	try_files $uri $uri/ /index.php$is_args$query_string;  
  }  
  ```
* 安装php扩展
  `fileinfo`或者需要通过宝塔面板安装,命令行安装失败
  {% tabs, phpDep %}
  <!-- tab Centos @linux -->
  ``` bash
  yum install php-zip php-xml php-curl php-openssl php-mbstring php-bcmath php-tokenizer php-ctype php-pdo php-json -y
  ```
  <!-- endtab -->
  <!-- tab Debian/Ubuntu @linux -->
  ``` bash
  apt install php-zip php-xml php-curl php-openssl php-mbstring php-bcmath php-tokenizer php-ctype php-pdo php-json -y
  ```
  <!-- endtab -->
  {% endtabs %}

* 安装依赖
  ``` bash
  php composer.phar install
  ```
  如果安装失败，出现以下错误请先更新`composer.json`配置文件
  ``` xml
  - The requested package youzan/open-sdk ^1.0 exists as youzan/open-sdk[2.0.0, 2.0.1, 2.0.10, 2.0.11...
  ```
  > 如果出现以下错误,需要禁用`putenv`函数
  ```
  [ErrorException]                                 
  putenv() has been disabled for security reasons
  ```
  ``` bash
  find / -name "php.ini" #查看php.ini文件位置
  sed -i 's/,putenv//g' /www/server/php/71/etc/php.ini
  ```
* 更新`composer.json`配置文件
  主要是更新`open-sdk`版本,从`"youzan/open-sdk": "^1.0"`改为`"youzanyun/open-sdk": "^2.0",`,
  ``` bash
cat <<EOT > composer.json
{
    "name": "laravel/laravel",
    "description": "The Laravel Framework.",
    "keywords": ["framework", "laravel"],
    "license": "MIT",
    "type": "project",
    "require": {
        "php": "^7.1.3",
        "barryvdh/laravel-ide-helper": "^2.5",
        "fideloper/proxy": "^4.0",
        "guzzlehttp/guzzle": "^6.3",
        "ipip/db": "^1.0",
        "itbdw/ip-database": "^2.0",
        "jenssegers/agent": "^2.6",
        "laravel/framework": "5.6.*",
        "laravel/tinker": "^1.0",
        "mews/captcha": "^2.2",
        "mews/purifier": "^2.1",
        "misechow/geetest": "^1.0",
        "misechow/no-captcha": "^1.0",
        "openlss/lib-array2xml": "^0.5.1",
        "overtrue/laravel-lang": "^3.0",
        "phpoffice/phpspreadsheet": "^1.6",
        "predis/predis": "^1.1",
        "rap2hpoutre/laravel-log-viewer": "^1.0",
        "riverslei/payment": "*",
        "spatie/laravel-permission": "^2.29",
        "youzanyun/open-sdk": "^2.0",
        "tymon/jwt-auth": "1.0.0-rc.2"
    },
    "require-dev": {
        "filp/whoops": "^2.0",
        "fzaninotto/faker": "^1.4",
        "mockery/mockery": "^1.0",
        "nunomaduro/collision": "^2.0",
        "phpunit/phpunit": "^7.0"
    },
    "autoload": {
        "files": [
            "app/helpers.php"
        ],
        "classmap": [
            "database/seeds",
            "database/factories"
        ],
        "psr-4": {
            "App\\\\": "app/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\\\": "tests/"
        }
    },
    "extra": {
        "laravel": {
            "dont-discover": [
            ]
        }
    },
    "scripts": {
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate"
        ]
    },
    "config": {
        "preferred-install": "dist",
        "sort-packages": true,
        "optimize-autoloader": true
    },
    "minimum-stability": "dev",
    "prefer-stable": true
}
EOT    
  ```
* 修改环境配置文件
修改其中的数据库字段: `DB_DATABASE` `DB_USERNAME` `DB_PASSWORD`
``` bash
cat <<EOT > .env
APP_DEMO=false
APP_NAME=ssrpanel
APP_ENV=local
APP_KEY=
APP_DEBUG=false
APP_URL=http://localhost
APP_TIMEZONE=Asia/Shanghai
APP_LOCALE=zh-CN
APP_FALLBACK_LOCALE=en
LOG_CHANNEL=daily

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ssrpanel
DB_USERNAME=ssrpanel
DB_PASSWORD=ssrpanel
DB_STRICT=false

BROADCAST_DRIVER=log
CACHE_DRIVER=file
SESSION_DRIVER=file
SESSION_LIFETIME=120
QUEUE_DRIVER=database

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.exmail.qq.com
MAIL_PORT=465
MAIL_USERNAME=admin@ssrpanel.com
MAIL_PASSWORD=password
MAIL_ENCRYPTION=ssl
MAIL_FROM_ADDRESS=admin@ssrpanel.com
MAIL_FROM_NAME=SSRPanel
MAILGUN_DOMAIN=
MAILGUN_SECRET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

REDIRECT_HTTPS=false

GEETEST_ID=
GEETEST_KEY =
NOCAPTCHA_SECRET=
NOCAPTCHA_SITEKEY=
EOT
```
* 设置权限

  ``` bash
  chown -R www:www storage/ && chmod -R 777 storage/
  ```
* 从[db.sql](https://raw.githubusercontent.com/ssrpanel/SSRPanel/master/sql/db.sql)内倒入数据库
  * 回到宝塔面板->数据库->添加数据库
    * 数据库名: ssrpanel
    * 数据库编码: utf8mb4
    * 用户名: ssrpanel
    * 密码: ssrpanel
    * 访问权限: 所有人
    * 提交

  ``` bash
  mysql -u ssrpanel -p ssrpanel < ./sql/db.sql
  ```
* 修改database.php 中的数据库字段
``` bash
cat <<EOT > config/database.php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Default Database Connection Name
    |--------------------------------------------------------------------------
    |
    | Here you may specify which of the database connections below you wish
    | to use as your default connection for all database work. Of course
    | you may use many connections at once using the Database library.
    |
    */

    'default' => env('DB_CONNECTION', 'mysql'),

    /*
    |--------------------------------------------------------------------------
    | Database Connections
    |--------------------------------------------------------------------------
    |
    | Here are each of the database connections setup for your application.
    | Of course, examples of configuring each database platform that is
    | supported by Laravel is shown below to make development simple.
    |
    |
    | All database work in Laravel is done through the PHP PDO facilities
    | so make sure you have the driver for your particular database of
    | choice installed on your machine before you begin development.
    |
    */

    'connections' => [

        'sqlite' => [
            'driver' => 'sqlite',
            'database' => env('DB_DATABASE', database_path('database.sqlite')),
            'prefix' => '',
        ],

        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'ssrpanel'),
            'username' => env('DB_USERNAME', 'ssrpanel'),
            'password' => env('DB_PASSWORD', 'ssrpanel'),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => env('DB_STRICT', true),
            'engine' => null,
        ],

        'pgsql' => [
            'driver' => 'pgsql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '5432'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
            'schema' => 'public',
            'sslmode' => 'prefer',
        ],

        'sqlsrv' => [
            'driver' => 'sqlsrv',
            'host' => env('DB_HOST', 'localhost'),
            'port' => env('DB_PORT', '1433'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
        ],

    ],

    /*
    |--------------------------------------------------------------------------
    | Migration Repository Table
    |--------------------------------------------------------------------------
    |
    | This table keeps track of all the migrations that have already run for
    | your application. Using this information, we can determine which of
    | the migrations on disk haven't actually been run in the database.
    |
    */

    'migrations' => 'migrations',

    /*
    |--------------------------------------------------------------------------
    | Redis Databases
    |--------------------------------------------------------------------------
    |
    | Redis is an open source, fast, and advanced key-value store that also
    | provides a richer set of commands than a typical key-value systems
    | such as APC or Memcached. Laravel makes it easy to dig right in.
    |
    */

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

];
EOT
```

* 生成key
  ``` bash
  php artisan key:generate
  ```
  如果生成key报`Class 'Tymon\JWTAuth\Providers\LaravelServiceProvider' not found`错误,执行以下命令
  ``` bash
  rm -rf ./bootstrap/cache/services.php
  rm -rf ./bootstrap/cache/config.php
  php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
  ```
  如果报错`Class 'Telegram\Bot\Laravel\TelegramServiceProvider' not found `,把改类給注释掉
  ``` bash
  sed -i  -e 's/Telegram\\Bot\\Laravel\\TelegramServiceProvider::class/\/\/Telegram\\Bot\\Laravel\\TelegramServiceProvider::class/g' config/app.php
  ```

  ## 搭建后段
  * 编译安装libsodium是服务器支持更多的加密算法
    ``` bash
    wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz #官网下载libsodium
    tar xzvf LATEST.tar.gz #解压
    cd libsodium* #切换目录
    ./configure #生成配置文件
    make -j8 && make install #编译安装
    echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf #添加运行库位置
    ldconfig 加载运行库
    ```
  * 配置shadowsocksrr服务端
    ``` bash
    cd /root # 切换到根目录
    git clone -b manyuser https://github.com/shadowsocksrr/shadowsocksr.git # 克隆源码
    cd shadowsocksr # 切换到目录
    ./setup_cymysql.sh #安装依赖
    ./initcfg.sh #初始化配置文件
    sed -i  -e 's/sspanelv2/glzjinmod/g' userapiconfig.py #修改接口类型
    ```
  * 修改ssr配置文件
``` bash
cat <<EOT > user-config.json
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,

    "password": "m",
    "method": "aes-128-ctr",
    "protocol": "auth_aes128_md5",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth_compatible",
    "obfs_param": "",
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": ["*:443#127.0.0.1:2333"], //2223 is caddy port
    "fast_open": false
}
EOT
```

  * 修改后段数据库配置文件
  ``` bash
cat <<EOT > usermysql.json
{
    "host": "127.0.0.1",
    "port": 3306,
    "user": "ssrpanel",
    "password": "ssrpanel",
    "db": "ssrpanel",
    "node_id": 1,
    "transfer_mul": 1.0,
    "ssl_enable": 0,
    "ssl_ca": "",
    "ssl_cert": "",
    "ssl_key": ""
}
EOT
  ```
  * 前台运行，如果运行正常，日志正常，表示没有问题,确定没有问题后`CTRL` + `C`取消前台运行
  ``` bash
  python server.py
  ```
  * 后台运行
  ``` bash
  ./run.sh
  ```
  * 停止
  ``` bash
  ./stop.sh
  ```

## caddy实现伪装流量
* 修改ssrpanel前端默认端口号，不要让他监听80,因为caddy需要监听80端口
* 安装caddy
  ``` bash
  curl https://getcaddy.com | bash -s personal
  ```
* 创建caddy的目录
  ``` bash
  mkdir -p /usr/local/caddy/ && cd /usr/local/caddy/
  ```
* 创建caddy配置文件
``` bash
echo "https://ssrpanel.duyuanchao.me:2333 {
 gzip
 tls duyuanchao.me@gmail.com
 proxy / https://www.baidu.com
}" > /usr/local/caddy/Caddyfile
```
* 后台运行caddy
``` bash
nohup caddy &
```
