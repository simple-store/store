### koa面向对象式编程

> 最近由于在用egg.js，所以十分羡慕它里面面向对象的逻辑，
>
> 自己参照着简单封装了一个面向对象的koa2-node的 写法

- route层

  > 定义了一堆页面路由
  >
  > - 用户请求对应的路由
  > - 进入对应的server.action方法

  - 文件目录

   ![image-20180816173405008](/Users/wuyunhe/Desktop/1.png)

  - admin/index.js

  ```js
  const model = require('../model') // 获取基本路由
  // 模块导出
  module.exports = {
    ...model,
  }
  
  ```

  - model/index.js

  ```js
  
  // enum http function: post、get
  const methods = require('../methods')
  
  module.exports = {
    // search
    'list': { method: methods.get },
    // add
    'insert': { method: methods.post },
    // update
    'update': { method: methods.post },
    // delete
    'delete': { method: methods.post },
  }
  
  ```

  - method.js

  ```js
  module.exports = {
    get: 'get',
    post: 'post',
  }
  ```

  - index.js

  ```js
  // 使用koa-router middleware
  const router = require('koa-router')()
  
  // 获取对应的service层
  const services = {
    admin: require('../services/AdminService'),
  }
  // 配置所有的routes文件
  const routes = (config => {
  	return config.reduce((copy, name) => {
      const obj = require(`./${name}`) // 获取对应个单个文件  admin.js etc..
      const newArr = Object.keys(obj).reduce((total, each) => {
        // 配置path,method etc..
        // {  path: '/api/admin/list', method: post, action: list, server: 'admin' }
        let item = { path: `/api/${name.toLowerCase()}/${each}`, method: obj[each].method, action: each, service: name }
        total.push(item)
        return total
      }, [])
      copy = copy.concat(newArr)
  	  return copy
  	}, [])
  })(Object.keys(services))
  
  // 配置最终的路由，形式为
  // router.get(url, service.action)
  
  routes.forEach(item => {
    const { method, path, service: serviceName, action } = item
    const service = services[serviceName]
    router[method](path, service[action])
  })
  module.exports = router
  
  ```

- service层

  > 对请求参数的操作层
  >
  > 对返回数据的加工层
  >
  > - 拿到request.body里面的参数
  > - 调用对应的controller
  > - 拿到返回的数据，加工返回

  - 文件目录

  ![image-20180816192505414](/Users/wuyunhe/Desktop/2.png)

  - BaseService.js

    ```js
    const { pojo,filterUnderLine,} = require('../helper') // 获取辅助类里面的一些方法
    const { success, failed, successWithCode } = pojo // 获取消息集里的一些辅助方法
    
    // 需要绑定this的方法
    const funcs = [
      'list',
      'insert',
      'update',
      'delete',
    ]
    class BaseService {
      constructor() {
        this.controller = null;
        // 循环遍历绑定this
        funcs.forEach(item => {
          this[item] = this[item].bind(this)
        })
    
      }
    
      // 查询方法
      async list(ctx) {
        // controller返回的是一个对象，success(成功为true， 失败为false), data(成功则有此数据), err(失败则有此对象)
        const { success: flag, data, err } = await this.controller.list()
        if (flag) {
          // success 为pojo消息集的成功返回
          ctx.body = success(
            data.map(item => 
              //  筛选下划线属性，返回驼峰
              filterUnderLine(item)
            )
          )
        } else {
          // failed 为pojo消息集的失败返回，下同
          ctx.body = failed(err)
        }
      }
      // 插入方法
      async insert(ctx) {
        const { row } = ctx.request.body
        const { success, err } = await this.controller.insert(row)
        if (success) {
          // successWithCode 为没有数据返回时的成功返回
          ctx.body = successWithCode('添加成功') // 没有数据则返回
        } else {
          // 同上
          ctx.body = failed(err)
        }
      }
      // 更新方法
      // 同上
      async update(ctx) {
        const { row } = ctx.request.body
        const { success, err } = await this.controller.update(row)
        if (success) {
          ctx.body = successWithCode('添加成功')
        } else {
          ctx.body = failed(err)
        }
      }
      // 删除方法
      // 同上
      async delete(ctx) {
        const { row } = ctx.request.body
        const { success, err } = await this.controller.delete(row)
        if (success) {
          ctx.body = successWithCode('添加成功')
        } else {
          ctx.body = failed(err)
        }
      }
    }
    
    module.exports = BaseService
    
    ```

  - AdminService.js

    ```js
    // 导入基类
    const BaseService = require('./BaseService')
    // 导入对应的controller
    const Controller = require('../controller/AdminController')
    // 生成一次controller
    const AdminController = new Controller()
    class AdminService extends BaseService {
      constructor() {
        super()
        // 绑定对应的controller
        this.controller = AdminController;
      }
    }
    module.exports = new AdminService()
    
    ```

    

- controller层

  > 对数据库的操作层
  >
  > - 接收service层传来的参数
  > - 对参数进行处理
  > - 执行sql语句，返回数据库结果

  - 文件目录

   ![image-20180816194020494](/Users/wuyunhe/Desktop/3.png)

  - BaseController.js

    ```js
    const pool = require('../lib/mysql') // 导入封装好的mysql库
    const { query } = pool // 导入query方法
    const { STATUS } = require('../enum') // 导入枚举类型STATUS
    const { NtNUpdate,filterCamel, } = require('../helper') // 导入helper内相关方法
    
    class BaseController {
      constructor() {
        this.table = ''
      }
      // 查询表内所有数据（非删除）
      async list() {
        const sql = `select * from ${this.table} where status != ?`
        return await query(sql, [STATUS.DELED])
      }
      // 插入新数据
      async insert(row) {
        const {
          keys,
          vals
        } = filterCamel(row) // 将驼峰命令转换为下划线
        const names = keys.join(',') // 对应的参数
        const questions = keys.map(item => '?').join(',') // 对应的参数占位符
        // 补全sql语句 insert into table (x, xx) values(x, xx)
        const sql = `insert into ${this.table}(${names},create_time,status) values(${questions},now(),?)`
        return await query(sql, [...vals, STATUS.NORMAL])
      }
    
      async update(row) {
        const {
          id,
        } = row; // 获取数据内的id
        // 删除id
        delete row.id
        // 启始sql
        let _sql = `update ${this.table} set `
        const {
          sql,
          args
        } = NtNUpdate(row, _sql)// 获取对象内非空值，加工sql 语句  update table set name=?, val= ?
        // 补全sql语句 update table set name = ?, val = ? where id = ?
        _sql = sql + 'where id = ?'
        return await query(_sql, [...args, id])
      }
    
      async delete(row) {
        const {
          id
        } = row // 获取数据内的id
        // 补全sql update table set status = ? where id = ?
        const sql = `update ${this.table} set status = ? where id = ?`
        return await query(sql, [STATUS.DELED, id])
      }
    
      excute(sql, vals) {
        // 执行方法
        return await = query(sql, vals)
      }
      // log 方法
      log({func, err}) {
        console.log(`excute function[${func}] occured error : ${err.message || err}`)
      }
    
    }
    
    module.exports = BaseController
    ```

  - AdminController

    ```js
    const BaseController = require('./BaseController') // 获得基类
    
    class AdminController extends BaseController {
      constructor() {
        super();
        this.table = 'tour_admin' // 赋值table
      }
    }
    module.exports = AdminController
    ```

- helper类

  - helper/index.js

    ```js
    const pojo = require('./pojo')
    
    /**
     * 
     * @param {Object} params  参数对象
     * @param {String} sql sql语句
     * @description 根据参数对象去改变sql语句，最后返回对应的sql语句
     * @return 返回处理后的sql语句
     */
    const update = (params, sql) =>  {
      let keys = Object.keys(params)
      let arr = []
      keys.forEach((key) => {
        if (key) {
          sql = sql + `${key} = ? ,`
          arr.push(params[key])
        }
      })
      sql = sql.substring(0, sql.length - 1)
      return {
        args: arr,
        sql,
      }
    }
    /**
     * 
     * @param {String} val  原下划线值
     * @param {String} char 要替换的字符
     * @description 根据原key去替换下划线后转为驼峰
     * @return 返回处理后的key
     */
    const replaceUnderLine = (val, char = '_') => {
      const arr = val.split('')
      const index = arr.indexOf(char)
      arr.splice(index, 2, arr[index+1].toUpperCase())
      val = arr.join('')
      return val
    }
    
    /**
     * 
     * @param {String} val  原下划线值
     * @description 下划线转驼峰
     * @return 返回处理后的key
     */
    const underline2Camel = (val) => {
      return val.replace(/\_(\w)/g, (all, letter) => {
        return letter.toUpperCase()
      })
    }
    
    /**
     * 
     * @param {String} val  原key
     * @param {String} char  要替换的字符
     * @description 驼峰转下划线
     * @return 返回处理后的key
     */
    const camel2UnderLine = (val, char = '_') => {
      return val.replace(/([A-Z])/g,`${char}$1`).toLowerCase();
    }
    
    /**
     * 
     * @param {Object} obj  原对象
     * @param {String} char  要替换的字符
     * @description 对象驼峰转下划线
     * @return 返回处理后的keys 和 对应的vals { aboutExample: 'example' }, ['about_example'], ['example']
     * @return { Array } keys 
     * @return { Array } vals 
     */
    const fileterCamel = (obj, char = '_') => {
      const keys = Object.keys(obj)
      return keys.reduce((init, item) => {
        const str = item
        if (~item.indexOf(char)) {
          str = camel2UnderLine(item)
        }
        init.keys.push(str)
        init.vals.push(obj[item])
        return init
      }, {
        keys: [],
        vals: [],
      })
    }
    
    /**
     * 
     * @param {Object} obj  原对象
     * @param {String} char  要替换的字符
     * @description 对象下划线转驼峰
     * @return 返回处理后的对象 { about_example: 'example' } { aboutExample: 'example' }
     * @return { Object } obj  
     */
    const  filterUnderLine = (obj, char = '_') => {
      const arr =  Object.keys(obj).filter(item => ~item.indexOf(char))
      arr.forEach(item => {
        const val = obj[item]
        const key = underline2Camel(item)
        obj[key] = val
        delete obj[item]
      })
      return obj
    }
    
    module.exports = {
      NtNUpdate: update,
      filterUnderLine,
      replaceUnderLine,
      replaceCamel: fileterCamel,
      pojo,
    }
    
    ```

  - helper/pojo.js

    ```js
    // 成功返回
    const success = (result) => {
      return {
        retCode: 200,
        retValue: result
      }
    }
    // 成功没数据返回
    const successWithCode = msg => {
      return {
        retCode: 200,
        msg,
      }
    }
    
    // 失败返回
    const failed = (error) => {
      console.log(error)
      return {
        retCode: 500,
        msg: error.message || error || '服务器异常'
      }
    }
    
    
    
    module.exports = {
      success,
      failed,
      successWithCode,
    }
    
    ```

- enum枚举

  - enum/index.js

    ```js
    const status = require('./status') // 获取状态
    const type = require('./type')// 获取类型
    
    // 导出
    module.exports = {
      STATUS: status,
      TYPES: type,
    }
    
    ```

  - enum/status.js

    ```js
    module.exports = {
      DELED: 404, // 删除状态
      NORMAL: 0, // 普通状态
    }
    ```

  - enum/type.js

    ```js
    module.exports = {
      MANAGER: 999, // 管理员
      USER: 0, // 用户
    }
    ```

- lib

  - lib/mysql.js

    ```js
    const mysql = require('mysql');
    const config = require('../config/default.js')
    
    // 连接池
    const pool  = mysql.createPool({ 
      host     : config.database.HOST, // 数据库连接端口
      user     : config.database.USERNAME, // 数据库用户名
      password : config.database.PASSWORD, // 数据库密码
      database : config.database.DATABASE // 数据库名称
    });
    
    /**
     * 
     * @param sql 接收的sql语句
     * @param {Array} values sql语句参数
     * @return { Object } { success: boolean, err || data  }
     */
    const query = function( sql, values ) {
      return new Promise(( resolve, reject ) => {
        pool.getConnection(function(err, connection) {
          if (err) {
            resolve( {
              success: false,
              err,
            } )
          } else {
            connection.query(sql, values, ( err, rows) => {
              if ( err ) {
                resolve({
                  success: false,
                  err,
                } )
              } else {
                resolve( {
                  success: true,
                  data: rows,
                } )
              }
              connection.release()
            })
          }
        })
      })
    }
    module.exports={
      query,
    }
    
    ```

- config

  - config/default.js

    ```js
    const config = {
      // 启动端口
      port: 1200,
      // 数据库配置
      database: {
        DATABASE: 'xxx',
        USERNAME: 'root',
        PASSWORD: 'xxx',
        PORT: 'xxx',
        HOST: 'xxx'
      }
    }
    
    module.exports = config
    
    ```

- src/index.js

  ```js
  var Koa=require('koa');
  var path=require('path')
  var bodyParser = require('koa-bodyparser');
  var session = require('koa-session-minimal');
  var MysqlStore = require('koa-mysql-session');
  var config = require('./config/default.js');
  var koaStatic = require('koa-static')
  var app=new Koa()
  const routers = require('./routes/index')
  
  
  
  // session存储配置
  const sessionMysqlConfig= {
    user: config.database.USERNAME,
    password: config.database.PASSWORD,
    database: config.database.DATABASE,
    host: config.database.HOST,
  }
  
  // 配置session中间件
  app.use(session({
    key: 'USER_SID',
    store: new MysqlStore(sessionMysqlConfig)
  }))
  
  // 配置跨域
  app.use(async (ctx, next) => {
    ctx.set('Access-Control-Allow-Headers', 'Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With')
    ctx.set('Access-Control-Allow-Origin', 'http://localhost:9000');
    ctx.set('Access-Control-Allow-Methods', 'PUT,DELETE,POST,GET');
    ctx.set('Access-Control-Allow-Credentials', true);
    ctx.set('Access-Control-Max-Age', 3600 * 24);
    await next();
  });
  // 配置静态资源加载中间件
  app.use(koaStatic(
    path.join(__dirname , './public')
  ))
  
  // 使用表单解析中间件
  app.use(bodyParser())
  
  // 使用新建的路由文件
  app.use(routers.routes()).use(routers.allowedMethods())
  
  // 监听在1200
  app.listen(config.port)
  
  console.log(`listening on port ${config.port}`)
  
  ```

  



到这里项目就可以完整跑起来了，

最后奉上项目地址：