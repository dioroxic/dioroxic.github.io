---
title: "php 多版本安装扩展"
categories:
  - php
tags:
  - php
  - php extension
date: 2021-11-17
---
因为项目中有使用imagick扩展所以需要在homestead环境中安装imagick扩展，
但是因为是多版本php所以在安装扩展的过程中遇到一些问题，所以记录一下多版本php安装扩展的踩坑过程

# 下载扩展源码包
首先去pecl官网去下载所需要安装扩展包，这里以imagick扩展为例
```
wget https://pecl.php.net/get/imagick-3.6.0RC2.tgz
```

# 进行解压
```
tar -zxvf imagick-3.6.0RC2.tgz
```

# 开始编译安装
解压完成后，cd进入文件夹
```
cd imagick-3.6.0RC2
```

首先需要phpize（如果没有安装，则须安装php-dev）默认homestead是安装了的所以不需要再安装一遍
```
phpize && ./configure –with-php-config=/usr/bin/php-config7.4（指定版本的php-config地址）
```

这里./configure可能会报错，信息为configure: error: not found. Please provide a path to MagickWand-config or Wand-config program.
<p></p>

解决方案为
```
sudo apt-get install build-essential
sudo apt-get install libmagickcore-dev libmagickwand-dev
```

然后进行编译安装
```
sudo make && sudo make install
```

没问题的话就接着做以下操作
```
cd /etc/php/你想要安装的php版本/mods-available/

sudo vim imagick.ini
填入：'extension=imagick.so' 并保存

cd /etc/php/你想要安装的php版本/fpm/conf.d
sudo ln -s /etc/php/你想要安装的php版本/mods-available/imagick.ini 20-imagick.ini
```

# 重启php-fpm
编译完成后需要重启fpm，首先查看php-fpm的master进程号
```
ps aux|grep php-fpm

root         697  0.0  1.1 257164 45608 ?        Ss   01:08   0:00 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
root         698  0.0  1.1 259996 46788 ?        Ss   01:08   0:00 php-fpm: master process (/etc/php/8.0/fpm/php-fpm.conf)
vagrant      828  0.0  0.4 260400 17552 ?        S    01:08   0:00 php-fpm: pool www
vagrant      830  0.0  0.4 260400 17556 ?        S    01:08   0:00 php-fpm: pool www
vagrant    16220  0.0  1.3 266276 55740 ?        S    03:04   0:00 php-fpm: pool www
vagrant    16221  0.0  1.4 267052 58592 ?        S    03:04   0:00 php-fpm: pool www
vagrant    17123  0.0  0.0   9028   672 pts/0    S+   03:32   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox php-fpm
```
然后kill主进程
```
kill -USR2 679
```
就可以了

# 参考的文章链接
https://blog.csdn.net/qq_16885135/article/details/78130281
https://blog.csdn.net/cpainter/article/details/53401831
https://blog.51cto.com/u_9025736/2372445
https://zhuanlan.zhihu.com/p/113112043