---
title: "php swoole websocket实战"
categories:
  - php
tags:
  - php
  - php extension
  - swoole
  - websocket
date: 2022-05-30
---
# 起因
在开发门票系统时，发现需要使用消息通知，通知客户端进场成功并关闭二维码

# 业务需求
门票系统分为两个端，员工端和客户端，客户订购了门票之后会给予一个二维码，
进场时需要出示二维码给员工进行扫码记录进场，员工扫完码之后需要通知客户端扫码成功并关闭二维码弹窗。

# 业务需求分析
因为在员工扫完码之后需要通知客户端进场成功并关闭弹窗，所以这个操作属于服务器主动推送消息，具体实现选用swoole来实现该业务要求。

# 具体代码实现（laravel 框架）
首先swoole的安装我就不再提了，官网上有具体的教程，如果你的服务器上有多个版本的php，安装扩展的方式可以参考我的php 多版本安装扩展这篇文章。

废话不多说直接上代码

首先我们需要创建一个command来作为一个常驻进程来处理websocket的请求
```
php artisan make:command WebSocket
```
创建完成之后，我们输入以下代码：
```
<?php


namespace App\Console\Commands;

use Illuminate\Console\Command;
use Swoole\Process;


class WebSocket extends Command
{
    private $ws;
    private $pid_file;
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'swoole {action}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'WebSocket';


    public function __construct()
    {
        parent::__construct();
        // 设置进程文件
        $this->pid_file =  __DIR__.'/../../../storage/swoole_websocket.pid';
    }

    public function handle()
    {
        $action = $this->argument('action');
        switch ($action) {
            case 'stop':
                $this->stop();
                break;
            case 'start':  //启动命令
                $this->start();
                break;
            default:
                $this->error('没有' . $action . '这个命令');
                break;
        }
    }

    public function start()
    {
        $pid = $this->getPid();
        $log = app('log')->channel('webSocket');

        //检测服务是否启动
        if ($pid && Process::kill($pid, 0)) {
            $this->error('服务已启动');
            exit;
        }

        //创建websocket服务器对象，监听0.0.0.0:9502端口
        $this->ws = new \swoole_websocket_server("0.0.0.0", 9502);

        $this->ws->set([
            'worker_num'=>8,
            'daemonize' =>1,
            'max_request'=>1000,
            'dispatch_mode'=>2,
            'pid_file' =>$this->pid_file,
        ]);

        //监听WebSocket连接打开事件
        $this->ws->on('open', function ($ws, $request) use ($log) {
            $log->debug($request->fd . "连接成功");
            // $ws->push($request->fd, "hello, welcome\n");
        });

        //监听WebSocket消息事件
        $this->ws->on('message', function ($ws, $frame) {
            //fd绑定客户端传过来的标识uid
            $ws->bind($frame->fd, $frame->data);
        });

        $this->ws->on('request', function ($request, $response) use ($log) {
            $data = $request->post['data'];  // 获取值
            // 循环已连接的客户，向指定客户推送消息
            foreach ($this->ws->connections as $fd) {
                // 检查连接是否为有效的 WebSocket 客户端连接
                if ($this->ws->isEstablished($fd)) {
                    // 获取连接信息
                    $clientInfo = $this->ws->connection_info($fd);
                    if (array_key_exists('uid', $clientInfo) && (int)$clientInfo['uid'] === (int)$request->post['uid']) {
                        $this->ws->push($fd, $data);
                    }
                }
            }
            $log->debug("client is PushMessage\n".$data);
        });

        //监听WebSocket连接关闭事件
        $this->ws->on('close', function ($ws, $fd) use ($log) {
            $log->debug("client:{$fd} is closed\n");
        });

        $this->ws->start();
    }

    public function stop()
    {
        if (!$pid = $this->getPid()) {
            $this->error('没有服务id');
            exit;
        }
        //关闭服务
        if (Process::kill($pid)) {
            $this->info('服务已停止');
            exit;
        }
        $this->info('停止服务失败');
    }

    // 获取进程pid
    private function getPid()
    {
        return file_exists($this->pid_file) ? file_get_contents($this->pid_file) : false;
    }

    public static function push($data)
    {
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, "http://127.0.0.1:9502");
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($curl, CURLOPT_HEADER, 1);
        curl_setopt($curl, CURLOPT_POST, 1);
        curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
        curl_exec($curl);
        curl_close($curl);
    }
}
```

开启常驻进程
```
php artisan swoole start
```

关闭常驻进程
```
php artisan swoole stop
``` 

扫码进场通知（伪代码）
```
  public function CodeScanningApproach()
  {
       // 逻辑代码......

       // 消息推送通知
       WebSocket::push(['data' => 'Successful mobilization', 'uid' => $user->getKey()]);
       return ['message' => 'success'];
  }
```