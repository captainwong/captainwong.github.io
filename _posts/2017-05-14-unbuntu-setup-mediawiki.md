---
layout: post
title:  "Ubuntu下部署MediaWiKi小记"
subtitle: "海纳百川，有容乃大"
date:   "2017-05-14" 
author: "cj"
tags:
    ubuntu
    mediawiki
    visualeditor
    parsoid
    postfix
---

## 一、MediaWiki简介

>MediaWiki是一款基于服务器的自由软件，并获得了 GNU 通用公共许可证 (GPL)的许可。这款软件的设计之目的是使其运行于一个每天拥有百万次点击量网站的大型服务器群组上。
>
>MediaWiki是一款极其强大、具有极强扩展性的软件，也是实现维基（wiki）丰富多彩的功能的一个载体，其使用PHP语言来处理和显示贮存在一个数据库中的数据，譬如MySQL。
>
>维基页面使用MediaWiki的维基文本格式编写，以便使那些不会使用HTML或CSS的用户也能轻松地编辑页面。
>
>当一个用户提交一个页面编辑时，MediaWiki就会将这次动作写入后台数据库，但不会删除之前页面的版本。因此，管理员就可以轻而易举地撤回编辑，以防破坏者或垃圾信息对页面的损坏。当然，MediaWiki也可以管理图片和其他流媒体文件，这些媒体文件贮存在档案系统之中。对于拥有大量用户的大型维基站点，MediaWiki支持高速缓存，亦可以轻松地与“鱿鱼代理服务器（Squid proxy server software）”软件联接。

没说的，一看是WikiPedia的后台程序，就它了！


## 二、安装MediaWiki

1. 搭建环境

腾讯云的虚拟主机，Ubuntu 16.04.2 TLS
apache2/mysql环境搭建就不提了，基本功。
因为我80端口另有用途，对外开放的站点都是用https，所以要设置Apache默认监听端口为443。

```
sudo vi /etc/apache2/ports.conf
Change Listen 80 to Listen 443
sudo vi /etc/apache2/sites-enabled/000-default.conf
Change <VirtualHost *:80> to <VirtualHost *:443>
```

安装php与其他依赖库

```
sudo apt-get update 
sudo apt-get upgrade
sudo apt-get install imagemagick php php7.0-intl php7.0-curl php7.0-gd php7.0-mbstring php7.0-mysql libapache2-mod-php php-xml php-mbstring
sudo a2enmod rewrite
sudo phpenmod mbstring
sudo phpenmod xml
```

2. 下载安装

访问[Download](https://www.mediawiki.org/wiki/Download)页面，下载最新版到/home/user/Downloads

```
wget https://releases.wikimedia.org/mediawiki/1.28/mediawiki-1.28.2.tar.gz
tar zxvf mediawiki-1.28.2.tar.gz
sudo mkdir /var/lib/mediawiki
sudo mv mediawiki-1.28.2/* /var/lib/mediawiki
```

3. 创建软链接并设置合适的权限

```
ln -s /var/lib/mediawiki /var/www/html/wiki
chown -R www-data:www-data /var/www/html/wiki
```


4. 创建数据库

```
mysql -u root -p
mysql> SET GLOBAL sql_mode='';
mysql> CREATE DATABASE wikidb;
mysql> CREATE USER 'wikiuser'@'localhost' IDENTIFIED BY 'y0uR-pa5sW0rd';
mysql> GRANT ALL PRIVILEGES ON wikidb.* TO 'wikiuser'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> quit
```

5. 在Apache中添加新虚拟主机的配置并重启Apache

```
touch /etc/apache2/sites-available/mediawiki.conf
ln -s /etc/apache2/sites-available/mediawiki.conf /etc/apache2/sites-enabled/mediawiki.conf
vi /etc/apache2/sites-available/mediawiki.conf
```

写入如下内容

```
<VirtualHost *:443>
ServerAdmin admin@your-domain.com
DocumentRoot /var/www/html/wiki/
ServerName your-domain.com
ServerAlias www.your-domain.com
<Directory /var/www/html/wiki/>
Options FollowSymLinks
AllowOverride All
Order allow,deny
allow from all
</Directory>
ErrorLog /var/log/apache2/your-domain.com-error_log
CustomLog /var/log/apache2/your-domain.com-access_log common
</VirtualHost>
```

重启Apache:

`sudo systemctl restart apache2`

6. 配置MediaWiki

访问https://your-domain.com/wiki
根据提示进行配置，Store Engine 选择 InnoDB， Database charactor set 选择 UTF-8，其他随意。

配置完成后下载LocalSettings.php并放入/var/www/html/wiki/，在尾部追加如下内容：

```
# Speed improvements
$wgUseGzip = true;
$wgUseFileCache = true;

# Performance settings
$wgDisableCounters = true;
$wgMiserMode = true;
```

现在可以访问https://your-domain.com/wiki 进行编辑了

7. 安装VisualEditor插件

访问[网址](https://www.mediawiki.org/wiki/Special:ExtensionDistributor?extdistname=VisualEditor&extdistversion=REL1_28)找到最新版VisualEditor下载地址。

```
wget https://extdist.wmflabs.org/dist/extensions/VisualEditor-REL1_28-93528b7.tar.gz
tar -xzf VisualEditor-REL1_28-93528b7.tar.gz -C /var/www/wiki/extensions
```

编辑/etc/www/html/wiki/LocalSettings.php，追加如下内容：

```
wfLoadExtension( 'VisualEditor' );
$wgDefaultUserOptions['visualeditor-enable'] = 1;
$wgHiddenPrefs[] = 'visualeditor-enable';
```

此时VisualEditor插件已经生效了，但只能新建词条并编辑，不能保存，不能编辑已有的词条。

8. 安装parsoid
parsoid依赖npm，首先安装npm

```
sudo apt-get install nodejs npm
```

下载parsoid到/home/user/Downloads并解压安装

```
git clone https://gerrit.wikimedia.org/r/p/mediawiki/services/parsoid
cd parsoid
npm install
```

配置parsoid

```
cp config.example.yaml config.yaml
```

编辑

```
uri: 'https://you-domain.com/wiki/api.php'
domain: 'localhost'  # optional
serverPort: 8000
serverInterface: '127.0.0.1'
```

注意由于启用了https，因此uri不能使用https://localhost/wiki/api.php ，会导致ssl验证失败无法访问。

运行

```
nohup npm bin/server.js &
```

测试

```
curl localhost:8000
```

使MediaWiki可以找到parsoid，编辑/var/www/html/wiki/LocalSettings.php，追加如下内容：

```
$wgVirtualRestConfig['modules']['parsoid'] = array(
        'url' => 'http://localhost:8000',
        'domain' => 'localhost',
        'prefix' => 'localhost'
);
```

## 三、搭建邮件服务器

由于安全原因，需要配置MediaWiki安全策略使得注册用户才可编辑，而用户注册需要验证邮箱，因此需要安装postfix。

以下皆以admin@your-domain.com为例说明。

1. 创建admin用户

```
sudo useradd -G root admin
passwd admin
su - admin
```

2. 安装

```
sudo DEBIAN_PRIORITY=low apt-get install postfix
```

在安装向导中作如下设置：

* General type of mail configuration?: Internet Site
* System mail name: your-domain.com (not mail.example.com)
* Root and postmaster mail recipient: admin
* Other destinations to accept mail for: $myhostname, your-domain.com, mail.your-domain.com, localhost.your-domain.com, localhost
* Force synchronous updates on mail queue?: No
* Local networks: 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
* Mailbox size limit: 0
* Local address extension character: +
* Internet protocols to use: all

如果有需要随时可以重新配置

```
sudo dpkg-reconfigure postfix
```

3. 配置

使用Maildir格式，基于用户操作会自动将邮件归档到不同的文件夹（cur, new, sent, tmp）内。另一种可用的格式是mbox，会将所有邮件保存为一个文件。

```
sudo postconf -e 'home_mailbox= Maildir/'
sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'
```

4. 映射邮件地址到Linux账户

```
sudo vi /etc/postfix/virtual
```

写入

```
contact@your-domain.com admin
admin@your-domain.com admin
```

应用并重启postfix

```
sudo postmap /etc/postfix/virtual
suto systemctl restart postfix
```

5. 添加防火墙例外

```
sudo ufw allow Postfix
```

我的虚拟主机是腾讯云，需要配置安全策略，添加邮件收发需要用到的端口到例外：

* SMTP - port 25 or 2525 or 587
* Secure SMTP (SSL / TLS) - port 465 or 25 or 587, 2526 (Elastic Email)
* POP3 - port 110
* IMAP - port 143
* IMAP SSL (IMAPS) - port 993

6. 设置环境变量

```
echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh
sudo source /etc/profile.d/mail.sh
```

7. 安装配置邮件客户端s-nail

```
sudo apt-get install s-nail
sudo vi /etc/s-nail.rc
```

追加如下内容：

```
set emptystart
set folder=Maildir
set record=+sent
```

8. 测试

   1. 给本机账户发一封邮件
    
```
echo 'hello world' | mail -s 'this is a subject' -Snorecord admin
```

此时会有如下错误

```
Output
Can't canonicalize "/home/admin/Maildir"
```

这是正常的且只会出现一次。可以使用

```ls -R ~/Maildir```

来确保目录存在。可以看到目录结构已经被创建，新的邮件已经被归档到~/Maildir/new中：

```
Output
/home/admin/Maildir/:
cur  new  tmp

/home/admin/Maildir/cur:

/home/admin/Maildir/new:
1463177269.Vfd01I40e4dM691221.mail.your-domain.com

/home/sammy/Maildir/tmp:
```

   2. 给外部用户如user@email.com发送一封邮件
   
```
echo 'Accorss the Greate Wall we can reach every corner in the world!' | mail -s 'Hello Wolrd!' -r admin 'user@email.com'
```

选项说明：

* -s: 邮件主题
* -r: 可选的发件人
* user@email.com 收件人

可以运行以下命令查看已发邮件

```
mail
file +sent
```

## 四、完工

打开https://your-domain.com/wiki ，注册一个账户，创建一个页面，尽情享受“海纳百川，有容乃大”吧！

Enjoy it!

## 参考资料

* [Manual:Running MediaWiki on Debian or Ubuntu](https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu)
* [How to install MediaWiki on an Ubuntu 16.04 VPS](https://www.rosehosting.com/blog/how-to-install-mediawiki-on-an-ubuntu-16-04-vps/)
* [Parsoid](https://www.mediawiki.org/wiki/Parsoid)
* [Parsoid/Setup](https://www.mediawiki.org/wiki/Parsoid/Setup)
* [How To Install and Configure Postfix on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-16-04)
* [How To Install Node.js on an Ubuntu 14.04 server](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server)
* [How To Install and Configure Postfix on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-16-04)
* [Common SMTP port numbers](http://docs.mailpoet.com/article/59-default-ports-numbers-smtp-pop-imap)

