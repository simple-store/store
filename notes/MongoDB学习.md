### MongoDB学习

> 由于最近像后台看齐，所以特意来学一下MongoDB

- 安装

  使用homebrew命令

  `brew install mongodb`

- 配置

    `mongod --config /usr/local/etc/mongod.conf`

- 启动

  新打开一个tab，输入`mongo`默认端口是27017,更换端口使用`--port` / `--host`参数

  之后便进入了mongodb的页面

- 测试

  `db.test.save( { a: 1 } )`

  控制台输出`WriteResult({ "nInserted" : 1 })`

  `db.test.find()`

  控制台输出`{ "_id" : ObjectId("5ba05adec8d190460f6706e8"), "a" : 1 }`

- 相关命令

  - 查看当前所有数据库: `show dbs`

  - 使用指定数据库 `use xxx`

  - 帮助 `help`

  - 插入 `insert`

  - 查询 `find(<query>, <projection>)`

    > 会返回一个游标，通过hasNext判断是否还有下一个，next拿到值并将指针指向下一行

    - query

      相当于sql中的where条件式

      ```
      find({
            _id: 5
         })
      ```

      ```
      $in
      ```

    - projection

      

  - 从集合返回单文档 `findOne`

  - 限制数量 `things.find().limit(row)`

- 

  