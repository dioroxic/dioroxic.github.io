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

# 环境变量
## 只对当前shell生效（shell脚本中常用）
方法一：
```
$PATH="$PATH":YOUR_PATH
```

方法二：

```
export PATH="$PATH:YOUR_PATH"
```
## 对所有用户所有shell都生效, 需要root权限（管理员常用）
方法一（修改environment文件）：

- 打开environment文件```vim /etc/environment```

- 修改PATH变量，在变量字符串末尾加:和 YOUR_PATH
```PATH="...:YOUR_PATH"```
- 使配置立即生效
```source /etc/environment```

```
如果设置后，系统重启后不可登录的解决方法：

在登录界面进入命令行模式：按组合键 alt +ctrl+f1
/usr/bin/sudo /usr/bin/vi /etc/environment
修改为正确配置，或者直接删除为空
保存退出后重启
```

方法二（修改profile文件）：

- 打开文件
```vim /etc/profile```

- 在打开的文件末添加
```export PATH ="$PATH:YOUR_PATH"```

## 只对某个用户的所有shell生效，只需要用户权限即可（用户常用）
- 打开设置文件
```vim ~/.bashrc```
- 在打开的文件末添加
```export PATH ="$PATH:YOUR_PATH"```
- 使文件配置立即生效
```source ~/.bashrc```