### koa封装路由

> 由于直接写router.get('/api/xxx/xxx')会产生许多冗余的代码，造成视觉上的不美观
>
> 所以本人秉着强迫症的原则去小小地封装了一下自己的路由
>
> 目录大致为routes(各个路由的定义)、services（服务层-数据中转层）、controller(对数据库的直接操作)、helper(封装了pojo消息集类)、...
>
> 技术栈采用的是koa2 + koa-router + mysql



- 入口文件

  > routers/index.js

  ```js
  /**
   * 整合所有子路由
   */
  
  const router = require('koa-router')()
  const routes = require('../routes')
  
  routes.forEach(item => {
    const service = require(`../services/${item.service}`)
    router[item.method](item.path, service[item.action])
  })
  module.exports = router
  ```

  >  routes入口文件

  ```js
  
  module.exports = (config => {
  	return config.reduce((copy, name) => {
      const obj = require(`./${name}`)
      const newArr = Object.keys(obj).reduce((total, each) => {
        let item = { path: `/api/${name}/${each}`, method: obj[each].method, action: each, service: name }
        total.push(item)
        return total
      }, [])
      copy = copy.concat(newArr)
  	  return copy
  	}, [])
  })([
    'admin',
    'user',
    'order',
    'shop',
    'ticket',
    'type',
  ])
  
  ```

  - 文件夹目录为

    - routes/index.js
    - routes/admin
    - routes/xxx

  - 各个模块代码为

    ```js
    const model = require('../model')
    module.exports = {
      ...model,
      'login': { method: methods.post },
    }
    
    ```

  - model文件为

    ```js
    const methods = require('../methods')
    
    module.exports = {
      'list': { method: methods.get },
      'add': { method: methods.post },
      'update': { method: methods.post },
      'del': { method: methods.post },
    }
    
    
    ```

    

  - methods文件为

    ```js
    module.exports = {
      get: 'get',
      post: 'post',
    }
    
    ```

- services文件

  > 写好了路由的定义以及相关的方法，那么对应的services为
  >
  > 在这里你可以对数据进行一些特殊的处理

  - 对应的模块文件services/admin

    ```js
    const controller = require('../../controller/admin')
    const model = require('../model')
    const m  = model([
      'list',
      'add',
      'update',
      'del',
    ], 'admin')
    
    
    const login = async ctx => {
      let res;
      try {
        await controller.login().then(result => {
          res = success(result)
        })
      } catch(err) {
        res = failed(err)
      }
      ctx.body = res
    }
    module.exports = {
      ...m,
      login,
    }
    ```

  - model文件

    ```js
    const pojo = require('../../helper/pojo')
    const { success, failed } = pojo
    module.exports = (config, file) => {
      const controller = require(`../../controller/${file}`)
    	return config.reduce((copy, name) => {
        copy[name] = async ctx => {
          let res;
          try {
            const val = ctx.request.body
            await controller[name](val).then(result => {
              res = success(result)
            })
          } catch(err) {
            res = failed(err)
          }
          ctx.body = res
        }
    	  return copy
    	}, {})
    }
    
    ```

  - pojo类

    > 对返回的数据进行一层外包装

    ```js
    const success = (result) => {
      return {
        retCode: 200,
        retValue: result
      }
    }
    const failed = (error) => {
      console.log(error)
      return {
        retCode: 500,
        msg: error.message || '服务器异常'
      }
    }
    
    module.exports = {
      success,
      failed
    }
    ```

- controller文件目录

  > 对数据库进行真正底层的操作

  -   controller/admin

  ```js
  const pool = require('../../lib/mysql')
  const { NtNUpdate } = require('../../helper')
  const STATUS = require('../../enum')
  const TYPES = {
    NORMAL: 0,
  }
  const { query } = pool
  // 新添管理员
  const add = (val) => {
    const { account, phone, password, name, creator } = val
    const _sql = 'insert into admin(account,phone,password,create_time,creator,name,type,status) values(?,?,?,now(),?,?,?,?);'
    return query( _sql, [ account, phone, password, creator, name,TYPES.NORMAL,STATUS.NORMAL] )
  }
  
  const login = (val) => {
    const { account, password } = val
    const _sql = 'select * from admin where account = ? and password = ? and status = ?'
    return query( _sql, [ account, password, STATUS.NORMAL ] )
  }
  
  // 更改管理员
  const update = (val) => {
    const { account, phone, password, name, type, id } = val
    const _sql = 'update admin set '
    const { sql, args } = NtNUpdate({ account, phone, password, name, type }, _sql)
    _sql = sql + 'where id = ?'
    return query( _sql, [...args, id] )
  }
  
  // 查询管理员
  const list = val => {
    const sql = 'select * from admin where status != ?'
    return query(sql, [ STATUS.DEL ])
  }
  
  // 删除管理员
  const del = val => {
    const { id } = val
    const sql = 'update admin set status = ? where id = ?'
    return query(sql, [ STATUS.DEL, id ])
  }
  
  module.exports = {
    add,
    list,
    update,
    del,
    login,
  }
  ```

- mysql文件

  ```js
  const mysql = require('mysql');
  const config = require('../config/default.js')
  
  const pool  = mysql.createPool({
    host     : config.database.HOST,
    user     : config.database.USERNAME,
    password : config.database.PASSWORD,
    database : config.database.DATABASE
  });
  
  const query = function( sql, values ) {
    return new Promise(( resolve, reject ) => {
      pool.getConnection(function(err, connection) {
        if (err) {
          console.log(err)
          resolve( err )
        } else {
          connection.query(sql, values, ( err, rows) => {
            if ( err ) {
              reject( err )
            } else {
              resolve( rows )
            }
            connection.release()
          })
        }
      })
    })
  }
  
  ```

  

#### 到这里为止你就可以有一个很清晰的目录和文件夹了

