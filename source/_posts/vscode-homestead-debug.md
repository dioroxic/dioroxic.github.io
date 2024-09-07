---
title: "vscode+homestead 配置xdebug"
categories:
- php
tags:
- php
- xdebug
- vscode
date: 2021-10-26
---
# 起因
因为开发中需要使用断点调试，所以记录下在vscode+homestead下开发如何使用xdebug断点调试

# vscode 安装所需要的插件
首先需要安装去vscode上安装一些调试所需要的插件

# 安装php Debug插件
![alt 安装php debug](/images/vscode_install_phpdebug.png)

# 配置vscode调试
> 第一步：点击运行和调试

![alt 点击运行和调试](/images/vscode_clike_debug.png)

> 第二步：点击显示所有自动调试配置，点击添加配置，选择php。
> 然后编辑launch.json

![alt 配置debug](/images/vscode_phpdebug_one.png)
![alt 配置debug](/images/vscode_phpdebug_two.png)

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        ...,
        {
             "name": "Listen for XDebug on Homestead",
             "type": "php",
             "request": "launch",
             "pathMappings": {
                "/home/vagrant/code": "D:/work/code"    // 此处按实际映射路径填写
            },
            "port": 9003
        }
     ]
}
```
设置debug[全局配置](https://code.visualstudio.com/docs/editor/debugging#_global-launch-configuration)需要将launch.json的内容放在setting.json里面去

> 第三步：运行debug开始调试

![alt 运行debug](/images/vscode_run_phpdebug.png)