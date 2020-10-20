---
title: Redis使用Lua脚本
date: 2020-10-20 09:52:42
tags:
- Lua
- Linux
- Redis
categories: 
- Redis
---

## EVAL 命令

Redis Eval 命令使用 Lua 解释器执行脚本。

```sh
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...] 
```

* EVAL 命令的关键字。
* luascript Lua 脚本。
* numkeys 指定的 Lua 脚本需要处理键的数量，其实就是 key数组的长度。
* key 传递给 Lua 脚本零到多个键，空格隔开，在 Lua 脚本中通过 KEYS[INDEX]来获取对应的值，其中1 <= INDEX <= numkeys。
* arg是传递给脚本的零到多个附加参数，空格隔开，在 Lua 脚本中通过ARGV[INDEX]来获取对应的值，其中1 <= INDEX <= numkeys。

注意：numkeys无论什么情况下都是必须的命令参数。

```sh
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> EVAL "return redis.call('GET',KEYS[1])" 1 hello
"world"
127.0.0.1:6379> EVAL "return redis.call('GET','hello')" 0
"world"
```

## call 函数和 pcall 函数

区别：
call 执行命令错误时会向调用者直接返回一个错误；
pcall 则会将错误包装为一个我们上面讲的 table 表。

```sh
127.0.0.1:6379> EVAL "return redis.call('no_command')" 0
(error) ERR Error running script (call to f_1e6efd00ab50dd564a9f13e5775e27b966c2141e): @user_script:1: @user_script: 1: Unknown Redis command called from Lua script 
127.0.0.1:6379>  EVAL "return redis.pcall('no_command')" 0
(error) @user_script: 1: Unknown Redis command called from Lua script
127.0.0.1:6379> 
```

## 值转换

Lua 脚本向 Redis 返回小数，那么会损失小数精度,转换为字符串则是安全的。

```sh
127.0.0.1:6379> EVAL "return 3.14" 0
(integer) 3
127.0.0.1:6379> EVAL "return tostring(3.14)" 0
"3.14"
127.0.0.1:6379> 
```

## 原子执行

Lua 脚本在 Redis 中是以原子方式执行的，在 Redis 服务器执行EVAL命令时，在命令执行完毕并向调用者返回结果之前，只会执行当前命令指定的 Lua 脚本包含的所有逻辑，其它客户端发送的命令将被阻塞，直到EVAL命令执行完毕为止。因此 LUA 脚本不宜编写一些过于复杂了逻辑，必须尽量保证 Lua 脚本的效率，否则会影响其它客户端。


## 脚本管理

### SCRIPT LOAD

加载脚本到缓存以达到重复使用，避免多次加载浪费带宽，每一个脚本都会通过 SHA 校验返回唯一字符串标识。需要配合 EVALSHA 命令来执行缓存后的脚本。

```sh
127.0.0.1:6379> SCRIPT LOAD "return 'hello'"
"1b936e3fe509bcbc9cd0664897bbe8fd0cac101b"
127.0.0.1:6379> EVALSHA 1b936e3fe509bcbc9cd0664897bbe8fd0cac101b 0
"hello"
```

### SCRIPT FLUSH

清除所有的脚本缓存。Redis并没有根据 SHA 来删除脚本缓存,所以在生产中一般不会再生产过程中使用该命令。

### SCRIPT EXISTS

以 SHA 标识为参数检查一个或者多个缓存是否存在。

```sh
127.0.0.1:6379> SCRIPT EXISTS 1b936e3fe509bcbc9cd0664897bbe8fd0cac101b
1) (integer) 1
127.0.0.1:6379> 
```

### SCRIPT KILL

终止正在执行的脚本。但是为了数据的完整性此命令并不能保证一定能终止成功。如果当一个脚本执行了一部分写的逻辑而需要被终止时，该命令是不凑效的。需要执行SHUTDOWN nosave在不对数据执行持久化的情况下终止服务器来完成终止脚本。

## 总结

* 务必对 Lua 脚本进行全面测试以保证其逻辑的健壮性，当 Lua 脚本遇到异常时，已经执行过的逻辑是不会回滚的。
* 尽量不使用 Lua 提供的具有随机性的函数，参见相关官方文档。
* 在 Lua 脚本中不要编写 function 函数,整个脚本作为一个函数的函数体。
* 在脚本编写中声明的变量全部使用 local 关键字。
* 在集群中使用 Lua 脚本要确保逻辑中所有的 key分 到相同机器，也就是同一个插槽 (slot) 中，可采用 Redis Hash Tag 技术。
* Lua 脚本一定不要包含过于耗时、过于复杂的逻辑。

参考：

[一网打尽Redis Lua脚本并发原子组合操作](https://mp.weixin.qq.com/s/hwa0tm1apVursY6KvB3FfA)

[Redis Eval 命令](https://www.runoob.com/redis/scripting-eval.html)