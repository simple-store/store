### Redis学习

> 最近在向node后台开发看齐，所以了解了一些关于redis的知识，这里作为笔记

- 安装redis

 使用home-brew

`brew install redis`

成功的提示信息

```shell
To have launchd start redis at login: 
ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents 
Then to load redis now: 
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist 
Or, if you don’t want/need launchctl, you can just run: 
redis-server /usr/local/etc/redis.conf
```

使用cat命令查看配置信息

`cat /usr/local/etc/redis.conf`

其中指定了端口等信息

```shell

bind 127.0.0.1 ::1
bind 127.0.0.1
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
...
```

- 启动redis

  `redis-server /usr/local/etc/redis.conf`

  终端输出

  ```shell
  9400:C 17 Sep 21:51:36.794 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  9400:C 17 Sep 21:51:36.795 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=9400, just started
  9400:C 17 Sep 21:51:36.795 # Configuration loaded
  9400:M 17 Sep 21:51:36.796 * Increased maximum number of open files to 10032 (it was originally set to 7168).
                  _._
             _.-``__ ''-._
        _.-``    `.  `_.  ''-._           Redis 4.0.11 (00000000/0) 64 bit
    .-`` .-```.  ```\/    _.,_ ''-._
   (    '      ,       .-`  | `,    )     Running in standalone mode
   |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
   |    `-._   `._    /     _.-'    |     PID: 9400
    `-._    `-._  `-./  _.-'    _.-'
   |`-._`-._    `-.__.-'    _.-'_.-'|
   |    `-._`-._        _.-'_.-'    |           http://redis.io
    `-._    `-._`-.__.-'_.-'    _.-'
   |`-._`-._    `-.__.-'    _.-'_.-'|
   |    `-._`-._        _.-'_.-'    |
    `-._    `-._`-.__.-'_.-'    _.-'
        `-._    `-.__.-'    _.-'
            `-._        _.-'
                `-.__.-'
  
  9400:M 17 Sep 21:51:36.799 # Server initialized
  9400:M 17 Sep 21:51:36.800 * DB loaded from disk: 0.001 seconds
  9400:M 17 Sep 21:51:36.800 * Ready to accept connections
  ```

  这里可以看到redis启动成功，监听的端口是6379

  新开一个tab窗口

  ` redis-cli ping`

  可以看到控制台输出`PONG`

  这就说明redis启动成功

- 关闭redis

  - 在启动窗口按cmd+c或者直接关掉窗口结束进程

  - 执行`redis-cli shutdown`

    输出`redis-cli ping`检验

    控制台打印`Could not connect to Redis at 127.0.0.1:6379: Connection refused`

    说明关闭成功



### node端的简单使用

- node安装redis

  `npm install redis --save `

- 启动redis客户端

  ```js
  const redis = require('redis');
  
  // 你启动的redis端口和ip地址+options（可去官网自行查阅资料）
  const client = redis.createClient(6379, '127.0.0.1', {});
  // 监听错误
  client.on('error', (err) => {
    console.log(err);
  })
  // 设置普通的string
  client.set("string key", "string val", redis.print);
  // 设置hash值
  client.hset("hash key", "hashtest 1", "some value", redis.print);
  client.hset(["hash key", "hashtest 2", "some other value"], redis.print);
  // 设置map值
  client.mset('key1', 'val1', ... 'keyn', 'valn', '[callback]'); 
  
  //循环遍历
  client.hkeys("hash key", function (err, replies) {
      console.log(replies.length + " replies:");
      replies.forEach(function (reply, i) {
          console.log("    " + i + ": " + reply);
      });
      // 退出
      client.quit();
  }); 
  ```

  更多命令使用请参考: <https://github.com/NodeRedis/node_redis>

- 漂流瓶小功能

  - throw方法

  ```js
  exports.throw = (bottle, callback) => {
    bottle.time = bottle.time || Date.now();
    const bottleId = Math.random().toString(16);
    const type = {
      male: 0,
      female: 1
    };
    console.log('at redis.js:', bottle);
    /*client.SELECT选择数据库编号*/
      client.SELECT(type[bottle.type], function (event) {
        console.log(event);
        /*client.HMSET 保存哈希键值*/
          client.HMSET(bottleId, bottle, function (err, result) {
            if (err) {
              return callback({
                code: 0,
                msg: "过会儿再来试试吧！"
              });
            }
            console.log('at redis.js:', result);
            callback({
              code: 1,
              msg: result
            });
            /*设置过期时间为1天*/
            client.EXPIRE(bottleId, 86400);
          });
      });
  }
  ```

  - pick方法

    ```js
    exports.pick = function (info, callback) {
      const type = {
        all: Math.round(Math.random()),
        male: 0,
        female: 1
      };
      console.log('info is:', info);
      info.type = info.type || 'all';
      client.SELECT(type[info.type], function () {
        /*随机返回当前数据库的一个键*/
        client.RANDOMKEY(function (err, bottleId) {
          if (!bottleId) {
            return callback({
              code: 0,
              msg: "大海空空如也..."
            });
          }
          /*根据key返回哈希对象*/
          client.HGETALL(bottleId, function (err, bottle) {
            if (err) {
              return callback({
                code: 0,
                msg: "漂流瓶破损了..."
              });
            }
            callback({
              code: 1,
              msg: bottle
            });
            /*根据key删除键值*/
            client.DEL(bottleId);
          });
        });
      });
    }
    ```

    

    - app.js

      ```js
      const redislib = require('./lib/redis');
      
      redislib.throw({
        type: 'male',
      }, res => {
        console.log(res);
      })
      redislib.pick({
        type: 'male',
      }, res => {
        console.log('at callback:', res);
      });
      ```

      

    到这里已经跑通了程序，尽情享受redis吧～