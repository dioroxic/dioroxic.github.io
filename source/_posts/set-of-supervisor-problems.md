---
title: "supervisor 问题集合"
categories:
- supervisor
tags:
- supervisor
date: 2023-11-30
---
## unix:///tmp/supervisor.sock no such file
先查看supervisord进程
``` bash
ps -ef | grep supervisord
```
会出现
```
root         718       1  0 Nov27 ?        00:01:56 /usr/bin/python3 /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
vagrant    70241   70188  0 09:32 pts/0    00:00:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox supervisord
```
然后kill supervisord进程
``` bash
kill -s SIGTERM 进程id
```