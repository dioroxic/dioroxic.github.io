# php
[xdebug]
xdebug.mode = coverage,debug,develop
xdebug.client_port = 9000
xdebug.discover_client_host = true
xdebug.max_nesting_level = 512

# mysql
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

## mysql8.0容器因为环境改变导致原始数据初始化的lower_case_table_names和现有环境不一致
mysql容器因为系统环境的改变导致lower_case_table_names（表名大小写敏感）不一致让mysql容器一直错误中止重启的问题

log 错误如下：

Different lower_case_table_names settings for server ('2') and data dictionary ('0').


| 参数值                     | 参数说明                       |
|-------------------------|----------------------------|
| lower_case_table_names=0 | 表名存储为给定的大小，比较时区分大小写的       |
| lower_case_table_names=1 | 表名存储在磁盘是小写的，但是比较的时候是不区分大小写 |
| lower_case_table_names=2 | 表名存储为给定的大小写，比较时小写为标准       |

**windows环境默认 1 ,linux环境默认0 ,macos环境默认2**

出现这个问题原因是：最开始时容器是在linux上存储和生成数据的（文件映射到宿主机），后期修改本地环境改到win上重新搭建容器从而导致老数据里存储的lower_case_table_names参数值和现有环境默认参数值不一致，让容器一直中止停止重启，解决方法很简单只需要删除调原有环境的里的mysql数据就行了（提前备份sql）。

因为mysql8.0里的lower_case_table_names是初始化时就设置好了，所以本地环境最快的解决方案就是备份好之前数据的sql然后删除掉原有数据映射目录里的数据就好了


# linux
## 宝塔
去除禁用函数
putenv
proc_open
pcntl_signal
pcntl_alarm
symlink

## 环境变量
### 只对当前shell生效（shell脚本中常用）
方法一：
```
$PATH="$PATH":YOUR_PATH
```

方法二：

```
export PATH="$PATH:YOUR_PATH"
```
### 对所有用户所有shell都生效, 需要root权限（管理员常用）
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

### 只对某个用户的所有shell生效，只需要用户权限即可（用户常用）
- 打开设置文件
  ```vim ~/.bashrc```
- 在打开的文件末添加
  ```export PATH ="$PATH:YOUR_PATH"```
- 使文件配置立即生效
  ```source ~/.bashrc```

# phpstorm wsl创建不了文件
sudo chown -R username /path/to/working/directory

ubuntu{$version} config --default-user root # 修改wsl ubuntu 默认用户

# wsl 权限问题
先设置默认为root，然后chown -R root:root /path/to/working/directory && chmod -R 777 /path/to/working/directory

