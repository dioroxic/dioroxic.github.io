---
title: 'laravel Facade是如何实现的'
date: 2024-07-24 13:58:57
---

## 什么是Facades
Laravel的Facades提供了一种便捷和简洁的方式来访问服务容器中的类和对象

## 具体实现
这里用Cache Facades类来做演示
```php
Cache::get()
```
当使用Cache::get()时，会先触发调用Cache的父类Facade下的__callStatic方法，下面是Facade下的__callStatic方法具体实现
```php
    /**
     * 处理对对象的动态静态调用。
     *
     * @param  string  $method
     * @param  array  $args
     * @return mixed
     *
     * @throws \RuntimeException
     */
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }
```
发现会先去调用getFacadeRoot方法来从容器所需要的对象实例，获取对象实例之后会调用传入方法。接下来看看getFacadeRoot方法的具体实现
```php
    Facade类
    /**
     * 获取 Facade 后面的根对象。
     *
     * @return mixed
     */
    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }

    /**
     * 获取组件的注册名称。
     *
     * @return string
     *
     * @throws \RuntimeException
     */
    protected static function getFacadeAccessor()
    {
        throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
    }
    
    /**
     * 从容器中解析 Facade 根实例。
     *
     * @param  object|string  $name
     * @return mixed
     */
    protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        if (static::$app) {
            return static::$resolvedInstance[$name] = static::$app[$name];
        }
    }
    
    
     Cache类
     /**
     * 获取组件的注册名称
     *
     * @return string
     */
    protected static function getFacadeAccessor()
    {
        return 'cache';
    }
```
发现会先去Cache类调用getFacadeAccessor类来获取组件注册在容器里的名称，然后如果Cache类没有实现这个方法的话则会抛出RuntimeException异常来说明Cache类没有设置组件名称，最后获取到在容器注册的名称之后会去调用resolveFacadeInstance方法来从容器中实例化名称为cache的类（resolveFacadeInstance方法中的static::$app具体是在Illuminate\Foundation\Http\Kernel中的RegisterFacades里赋值的）。最后从容器中拿到对象实例之后就会去调用传入的方法get。
