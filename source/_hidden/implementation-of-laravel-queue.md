---
title: "laravel队列实现"
hidden: true
categories:
- php
tag:
- laravel
---

## 生产者
查看TestJob::dispatch()方法
```
    /**
     * 使用给定参数调度作业
     *
     * @return \Illuminate\Foundation\Bus\PendingDispatch
     */
    public static function dispatch(...$arguments)
    {
        return new PendingDispatch(new static(...$arguments));
    }
```
发现会实例化PendingDispatch类并将当前调用的job实例化传入到PendingDispatch类

查看PendingDispatch类
```
    /**
     * 创建新的挂起作业调度
     *
     * @param  mixed  $job
     * @return void
     */
    public function __construct($job)
    {
        $this->job = $job;
    }
    
    
    /**
     * 处理对象的破坏
     *
     * @return void
     */
    public function __destruct()
    {
        if (! $this->shouldDispatch()) {
            return;
        } elseif ($this->afterResponse) {
            app(Dispatcher::class)->dispatchAfterResponse($this->job);
        } else {
            app(Dispatcher::class)->dispatch($this->job);
        }
    }
```
发现初始化类的时候会存储job到job属性，然后在对象摧毁的时候调用析构函数发放队列

查看Dispatcher发现是接口，查看继承找到Illuminate\Bus\Dispatcher类发现服务提供者是Illuminate\\Bus\\BusServiceProvider
```
use Illuminate\Contracts\Bus\Dispatcher as DispatcherContract;
use Illuminate\Contracts\Bus\QueueingDispatcher as QueueingDispatcherContract;

    /**
     * 注册服务提供商
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Dispatcher::class, function ($app) {
            return new Dispatcher($app, function ($connection = null) use ($app) {
                return $app[QueueFactoryContract::class]->connection($connection); // 返回队列驱动管理（根据config配置返回对应的队列驱动 RedisQueue、DatabaseQueue.......）
            });
        }); // 绑定单例到容器

        $this->registerBatchServices();

        $this->app->alias(
            Dispatcher::class, DispatcherContract::class
        ); // 设置别名意味着app(DispatcherContract::class)会解析拿出Dispatcher实例

        $this->app->alias(
            Dispatcher::class, QueueingDispatcherContract::class
        ); // 设置别名意味着app(QueueingDispatcherContract::class)会解析拿出Dispatcher实例
    }
``` 
找到对应接口实现后查看Illuminate\Bus\Dispatcher类，找到dispatch方法
```
    /**
     * 将命令调度到其相应的处理程序
     *
     * @param  mixed  $command
     * @return mixed
     */
    public function dispatch($command)
    {
    // 判断队列解析程序是否存在和任务是否继承ShouldQueue接口，true走队列，否则直接执行任务
        return $this->queueResolver && $this->commandShouldBeQueued($command)
                        ? $this->dispatchToQueue($command)
                        : $this->dispatchNow($command);
    }
```
然后查看dispatchToQueue方法
```
    /**
     * 将命令调度到队列后面的相应处理程序
     *
     * @param  mixed  $command
     * @return mixed
     *
     * @throws \RuntimeException
     */
    public function dispatchToQueue($command)
    {
        $connection = $command->connection ?? null; // 任务该用那个驱动默认null的话按照配置文件设置驱动来

        $queue = call_user_func($this->queueResolver, $connection);

        if (! $queue instanceof Queue) {
            throw new RuntimeException('Queue resolver did not return a Queue implementation.');
        }

        if (method_exists($command, 'queue')) {
            return $command->queue($queue, $command);
        }

        // 推送任务到队列
        return $this->pushCommandToQueue($queue, $command);
    }
```
发现会先获取队列驱动然后再根据驱动来推送任务（我这里是redis），然后查看pushCommandToQueue方法
```
    /**
     * 将命令推送到给定的队列实例上
     *
     * @param  \Illuminate\Contracts\Queue\Queue  $queue
     * @param  mixed  $command
     * @return mixed
     */
    protected function pushCommandToQueue($queue, $command)
    {
        if (isset($command->queue, $command->delay)) { // 判断任务是否设置了指定队列名称和延时时间
            return $queue->laterOn($command->queue, $command->delay, $command);
        }

        if (isset($command->queue)) { // 判断任务是否设置了指定队列名称
            return $queue->pushOn($command->queue, $command);
        }

        if (isset($command->delay)) { // 判断任务是否设置了延时时间
            return $queue->later($command->delay, $command);
        }

        return $queue->push($command);
    }
```
这里先看下push和later方法，这两个方法分别是没有设置延时时间的队列和设置了延时时间队列，两者推送的redis数据类型不一样前者用的是list后者用的是zset。

先看push也就是list类型的队列，查找push方法发现跳转的接口查看具体实现，有redis、数据库等等，我设置的是redis所以看redis，找到RedisQueue下的push方法
```
    /**
     * 将新作业推送到队列中
     *
     * @param  object|string  $job
     * @param  mixed  $data
     * @param  string|null  $queue
     * @return mixed
     */
    public function push($job, $data = '', $queue = null)
    {
        return $this->enqueueUsing(
            $job,
            $this->createPayload($job, $this->getQueue($queue), $data), // 序列化任务的一些信息
            $queue,
            null,
            function ($payload, $queue) { // 把任务放入队列的回调函数
                return $this->pushRaw($payload, $queue);
            }
        );
    }
```
push会先序列化任务的一些信息然后再把序列化的信息传给最后一个参数回调函数，查看pushRaw函数
```
    /**
     * 将原始负载推送到队列中
     *
     * @param  string  $payload
     * @param  string|null  $queue
     * @param  array  $options
     * @return mixed
     */
    public function pushRaw($payload, $queue = null, array $options = [])
    {
    // 执行lua脚本实现redis队列放入
        $this->getConnection()->eval(
            LuaScripts::push(), 2, $this->getQueue($queue),
            $this->getQueue($queue).':notify', $payload
        );

        return json_decode($payload, true)['id'] ?? null;
    }
    
    /**
     * 获取用于将作业推送到队列的 Lua 脚本
     *
     * KEYS[1] - The queue to push the job onto, for example: queues:foo
     * KEYS[2] - The notification list for the queue we are pushing jobs onto, for example: queues:foo:notify
     * ARGV[1] - The job payload
     *
     * @return string
     */
    public static function push()
    {
        return <<<'LUA'
-- Push the job onto the queue...
redis.call('rpush', KEYS[1], ARGV[1])
-- Push a notification onto the "notify" queue...
redis.call('rpush', KEYS[2], 1)
LUA;
    }

```
push 的流程就看完了大概就是先将任务信息序列化然后传给回调函数调用，回调函数调用pushRaw方法lua脚本发放任务。

接下来看later方法实现
```
    /**
     * 在延迟后将新作业推送到队列中
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  object|string  $job
     * @param  mixed  $data
     * @param  string|null  $queue
     * @return mixed
     */
    public function later($delay, $job, $data = '', $queue = null)
    {
        return $this->enqueueUsing(
            $job,
            $this->createPayload($job, $this->getQueue($queue), $data),
            $queue,
            $delay,
            function ($payload, $queue, $delay) {
                return $this->laterRaw($delay, $payload, $queue);
            }
        );
    }
    
    /**
     * 在延迟后将原始作业推送到队列中
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string  $payload
     * @param  string|null  $queue
     * @return mixed
     */
    protected function laterRaw($delay, $payload, $queue = null)
    {
        $this->getConnection()->zadd(
            $this->getQueue($queue).':delayed', $this->availableAt($delay), $payload
        );

        return json_decode($payload, true)['id'] ?? null;
    }
```
大致都跟push方法一样，唯一不同的是回调函数调用的是laterRaw方法而不是pushRaw方法

## 消费者

通过config/app.php providers数组找到ConsoleSupportServiceProvider控制台服务提供者，发现ArtisanServiceProvider找到queue:work命令注册方法registerQueueWorkCommand
```
    use Illuminate\Queue\Console\WorkCommand as QueueWorkCommand;

    /**
     * Register the command.
     *
     * @return void
     */
    protected function registerQueueWorkCommand()
    {
        $this->app->singleton('command.queue.work', function ($app) {
            return new QueueWorkCommand($app['queue.worker'], $app['cache.store']);
        });
    }
```
查看QueueWorkCommand类(也就是WorkCommand)
```
  public function handle()
    {
        // 是否是维护模式，是的话休眠
        if ($this->downForMaintenance() && $this->option('once')) {
            return $this->worker->sleep($this->option('sleep'));
        }

        // 监听打印队列状况
        $this->listenForEvents();

        // 队列连接名称
        $connection = $this->argument('connection')
                        ?: $this->laravel['config']['queue.default'];

        // 获取需要处理的队列名称
        $queue = $this->getQueue($connection);

        // 运行处理进程
        return $this->runWorker(
            $connection, $queue
        );
    }
```
查看runWorker方法
```
    public function __construct(Worker $worker, Cache $cache)
    {
        parent::__construct();

        $this->cache = $cache;
        $this->worker = $worker;
    }
    
    protected function runWorker($connection, $queue)
    {
        return $this->worker->setName($this->option('name'))
                     ->setCache($this->cache)
                     ->{$this->option('once') ? 'runNextJob' : 'daemon'}(
            $connection, $queue, $this->gatherWorkerOptions()
        );
    }
```
发现如果有带--once参数则走 Illuminate\Queue\Worker类下的runNextJob方法，没有则走daemon方法，并且传入队列连接名称、队列名称、当前命令参数。查看Illuminate\Queue\Worker下的daemon方法

首先会去判断是否能加载到pcntl扩展，能的话开始判断进程信号，然后再获取调用queue:restart时的时间戳
```
        // 加载pcntl扩展，然后判断进程信号
        if ($supportsAsyncSignals = $this->supportsAsyncSignals()) {
            /**
              * 当脚本被指示关闭时，会引发SIGTERM。
              * SIGUSR2是用户定义的信号，Laravel用来表示脚本应该暂停。
              * 当暂停的脚本继续进行时，会引发SIGCONT。
              * 这些信号从Process Monitor（如 Supervisor ）发送并与我们的脚本进行通信。
            */
            $this->listenForSignals();
        }
        
        // 获取调用queue:restart时的时间戳
        $lastRestart = $this->getTimestampOfLastQueueRestart();
```

最后开始循环获取并执行队列中的任务。开始循环时会先检查是否应该暂停脚本或退出杀死脚本
```
        // 检查是否应该暂停脚本
        if (! $this->daemonShouldRun($options, $connectionName, $queue)) {
            $status = $this->pauseWorker($options, $lastRestart);

            // 是否应该退出和杀死脚本进程
            if (! is_null($status)) {
                return $this->stop($status);
            }

            continue;
        }
```

之后会开始从队列中获取需要执行的任务
```
        // 首先，我们将尝试将下一个工作从队列中取出。我们还将注册超时处理程序并重置此作业的警报，以便它不会永远处于冻结状态。然后，我们可以完成这项工作。
        $job = $this->getNextJob(
            $this->manager->connection($connectionName), $queue
        );
```

查看getNextJob方法，该方法接收一个队列连接实例，和队列名称。
```        
    protected function getNextJob($connection, $queue)
    {
        $popJobCallback = function ($queue) use ($connection) {
            return $connection->pop($queue);
        };

        try {
            if (isset(static::$popCallbacks[$this->name])) {
                return (static::$popCallbacks[$this->name])($popJobCallback, $queue);
            }

            foreach (explode(',', $queue) as $queue) { // 根据队列名称取出任务
                if (! is_null($job = $popJobCallback($queue))) {
                    return $job;
                }
            }
        } catch (Throwable $e) {
            $this->exceptions->report($e);

            $this->stopWorkerIfLostConnection($e);

            $this->sleep(1);
        }
    }
```
查看connection->pop($queue)这里我是redis所以直接查看RedisQueue下的pop方法。先查看migrate，该方法会将延时或过期的队列推到list队列里面去
```
    $this->migrate($prefixed = $this->getQueue($queue));
    
    protected function migrate($queue)
    {
        // 将延时队列推送到list
        $this->migrateExpiredJobs($queue.':delayed', $queue);

        // 将过期队列推送到list
        if (! is_null($this->retryAfter)) {
            $this->migrateExpiredJobs($queue.':reserved', $queue);
        }
    }
```
然后查看migrateExpiredJobs方法具体实现
```
    public function migrateExpiredJobs($from, $to)
    {
        return $this->getConnection()->eval(
            LuaScripts::migrateExpiredJobs(), 3, $from, $to, $to.':notify', $this->currentTime()
        );
    }
```
发现执行了lua脚本，然后看看lua脚本具体实现
```
    /**
     * Get the Lua script to migrate expired jobs back onto the queue.
     *
     * KEYS[1] - The queue we are removing jobs from, for example: queues:foo:reserved
     * KEYS[2] - The queue we are moving jobs to, for example: queues:foo
     * KEYS[3] - The notification list for the queue we are moving jobs to, for example queues:foo:notify
     * ARGV[1] - The current UNIX timestamp
     *
     * @return string
     */
    public static function migrateExpiredJobs()
    {
        return <<<'LUA'
-- Get all of the jobs with an expired "score"...
local val = redis.call('zrangebyscore', KEYS[1], '-inf', ARGV[1])

-- If we have values in the array, we will remove them from the first queue
-- and add them onto the destination queue in chunks of 100, which moves
-- all of the appropriate jobs onto the destination queue very safely.
if(next(val) ~= nil) then
    redis.call('zremrangebyrank', KEYS[1], 0, #val - 1)

    for i = 1, #val, 100 do
        redis.call('rpush', KEYS[2], unpack(val, i, math.min(i+99, #val)))
        -- Push a notification for every job that was migrated...
        for j = i, math.min(i+99, #val) do
            redis.call('rpush', KEYS[3], 1)
        end
    end
end

return val
LUA;
    }
```
发现会将小于当前时间戳的成员从集合中取出来然后迁移到队列里面去，并且记录迁移的数量

```

```

```
    /**
     * Listen to the given queue in a loop.
     *
     * @param  string  $connectionName
     * @param  string  $queue
     * @param  \Illuminate\Queue\WorkerOptions  $options
     * @return int
    */
    public function daemon($connectionName, $queue, WorkerOptions $options)
    {
        while (true) {
          

            // First, we will attempt to get the next job off of the queue. We will also
            // register the timeout handler and reset the alarm for this job so it is
            // not stuck in a frozen state forever. Then, we can fire off this job.
            $job = $this->getNextJob(
                $this->manager->connection($connectionName), $queue
            );

            if ($supportsAsyncSignals) {
                $this->registerTimeoutHandler($job, $options);
            }

            // If the daemon should run (not in maintenance mode, etc.), then we can run
            // fire off this job for processing. Otherwise, we will need to sleep the
            // worker so no more jobs are processed until they should be processed.
            if ($job) {
                $jobsProcessed++;

                $this->runJob($job, $connectionName, $options);

                if ($options->rest > 0) {
                    $this->sleep($options->rest);
                }
            } else {
                $this->sleep($options->sleep);
            }

            if ($supportsAsyncSignals) {
                $this->resetTimeoutHandler();
            }

            // Finally, we will check to see if we have exceeded our memory limits or if
            // the queue should restart based on other indications. If so, we'll stop
            // this worker and let whatever is "monitoring" it restart the process.
            $status = $this->stopIfNecessary(
                $options, $lastRestart, $startTime, $jobsProcessed, $job
            );

            if (! is_null($status)) {
                return $this->stop($status);
            }
        }
    }
```