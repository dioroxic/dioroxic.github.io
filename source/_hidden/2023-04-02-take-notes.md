# php
[xdebug]
xdebug.mode = coverage,debug,develop
xdebug.client_port = 9000
xdebug.discover_client_host = true
xdebug.max_nesting_level = 512

# linux
## 宝塔
去除禁用函数
putenv
proc_open
pcntl_signal
pcntl_alarm
symlink

# mysql
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

# phpstorm wsl创建不了文件
sudo chown -R username /path/to/working/directory

ubuntu{$version} config --default-user root # 修改wsl ubuntu 默认用户

# wsl 权限问题
先设置默认为root，然后chown -R root:root /path/to/working/directory && chmod -R 777 /path/to/working/directory