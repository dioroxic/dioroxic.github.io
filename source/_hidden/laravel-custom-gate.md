---
title: 'laravel 自定义看守器'
date: 2024-07-24 13:58:57
---

## 前提
自定义看守器需要去实现Illuminate\Contracts\Auth\Guard这个契约

## 具体实现
这里以自定义jwt为看守器为例。首先创建JwtGuard类，然后继承并实现Illuminate\Contracts\Auth\Guard接口。可以先看看Guard接口有哪些方法
```php
namespace Illuminate\Contracts\Auth;

interface Guard
{
    /**
     * 确定当前用户是否已通过身份验证
     *
     * @return bool
     */
    public function check();

    /**
     * 确定当前用户是否为访客
     *
     * @return bool
     */
    public function guest();

    /**
     * 获取当前经过身份验证的用户
     *
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function user();

    /**
     * 获取当前经过身份验证的用户的ID
     *
     * @return int|string|null
     */
    public function id();

    /**
     * 验证用户的凭据
     *
     * @param  array  $credentials
     * @return bool
     */
    public function validate(array $credentials = []);

    /**
     * 设置当前用户
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable  $user
     * @return void
     */
    public function setUser(Authenticatable $user);
}
```
发现都定义了一些基础认证的方法，这里可以选择一个一个方法去实现也可以选择use GuardHelpers来实现看守器通用的代码，先来看看GuardHelpers实现的方法
```php
namespace Illuminate\Auth;

use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\UserProvider;

/**
 * These methods are typically the same across all guards.
 */
trait GuardHelpers
{
    /**
     * The currently authenticated user.
     *
     * @var \Illuminate\Contracts\Auth\Authenticatable
     */
    protected $user;

    /**
     * The user provider implementation.
     *
     * @var \Illuminate\Contracts\Auth\UserProvider
     */
    protected $provider;

    /**
     * Determine if the current user is authenticated. If not, throw an exception.
     *
     * @return \Illuminate\Contracts\Auth\Authenticatable
     *
     * @throws \Illuminate\Auth\AuthenticationException
     */
    public function authenticate()
    {
        if (! is_null($user = $this->user())) {
            return $user;
        }

        throw new AuthenticationException;
    }

    /**
     * Determine if the guard has a user instance.
     *
     * @return bool
     */
    public function hasUser()
    {
        return ! is_null($this->user);
    }

    /**
     * Determine if the current user is authenticated.
     *
     * @return bool
     */
    public function check()
    {
        return ! is_null($this->user());
    }

    /**
     * Determine if the current user is a guest.
     *
     * @return bool
     */
    public function guest()
    {
        return ! $this->check();
    }

    /**
     * Get the ID for the currently authenticated user.
     *
     * @return int|string|null
     */
    public function id()
    {
        if ($this->user()) {
            return $this->user()->getAuthIdentifier();
        }
    }

    /**
     * Set the current user.
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable  $user
     * @return $this
     */
    public function setUser(AuthenticatableContract $user)
    {
        $this->user = $user;

        return $this;
    }

    /**
     * Get the user provider used by the guard.
     *
     * @return \Illuminate\Contracts\Auth\UserProvider
     */
    public function getProvider()
    {
        return $this->provider;
    }

    /**
     * Set the user provider used by the guard.
     *
     * @param  \Illuminate\Contracts\Auth\UserProvider  $provider
     * @return void
     */
    public function setProvider(UserProvider $provider)
    {
        $this->provider = $provider;
    }
}
```
发现大部分Guard接口的方法都进行了实现，唯独validate和user方法没有实现。所以这里选用GuardHelpers的实现，所以代码将会是下面这样
```php
namespace App\Services\Auth;

use App\Services\JwtService;
use Illuminate\Auth\GuardHelpers;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Auth\UserProvider;
use Illuminate\Http\Request;

class JwtGuard implements Guard
{
    use GuardHelpers;

    protected JwtService $jwtService;
    protected Request $request;

    public function __construct(UserProvider $userProvider, Request $request, JwtService $jwtService)
    {
        $this->setProvider($userProvider);
        $this->request    = $request;
        $this->jwtService = $jwtService;
    }
    
    /**
     * 获取当前经过身份验证的用户
     *
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function user()
    {
        
    }

    /**
     * 验证用户的凭据
     * @param array $credentials
     * @return bool|void
     */
    public function validate(array $credentials = [])
    {
        
    }
}
```
初始化这里因为看守器需要对应的用户提供者来查询用户凭证所以需要传入UserProvider（这里使用的是EloquentUserProvider），然后因为jwt需要获取头部token之后做验证操作所以这里也需要传入Request和JwtService(自己实现的jwt生成和验证)。

做完上面操作之后，就可以开始去实现user和validate方法了，下面是user和validate方法的具体实现
```php
/**
     * 获取当前经过身份验证的用户
     *
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function user()
    {
        if (!is_null($this->user)) {
            return $this->user;
        }

        $user = null;
        if ($token = $this->request->bearerToken()) { // 查询Header的上带的token
            $user = $this->jwtService->verify($token); // 根据token来返回对应的用户
        }

        return $user;
    }

    /**
     * 验证用户的凭据
     * @param array $credentials
     * @return bool|void
     */
    public function validate(array $credentials = [])
    {
        // 通过给定的凭据检索用户
        $user   = $this->provider->retrieveByCredentials($credentials);
        return !is_null($user);
    }
```
这样需要实现的方法就实现好了但是没有获取登录用户token的方法，这里可以新增一个attempt方法来获取通过验证的用户的token，下面是代码实现。
```php
    public function attempt(array $credentials = [])
    {
        if ($user = $this->provider->retrieveByCredentials($credentials)) { // 通过给定的凭据检索用户
            $this->setUser($user);
            return $this->jwtService->generate($user);
        }
    }
```
这样就可以使用attempt方法来获取对应通过凭证用户的token了

## $request->user()是怎样拿到用户的？
查看源码会发现在AuthServiceProvider会去调用request下的setUserResolver方法并传入一个包含用户实例的闭包
```php
    /**
     * Handle the re-binding of the request binding.
     *
     * @return void
     */
    protected function registerRequestRebindHandler()
    {
        $this->app->rebinding('request', function ($app, $request) {
            $request->setUserResolver(function ($guard = null) use ($app) {
                return call_user_func($app['auth']->userResolver(), $guard);
            });
        });
    }
```