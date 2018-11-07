#### puppeteer爬虫扒取数据后存入数据库

> 由于最近的工作内容接触到了爬虫与测试
>
> 所以这里就记录了一个小小的例子

> 爬虫puppeteer + Koa2 + Mysql
>
> 是从之前koa2项目上增强了爬虫的功能
>
> 爬虫是以网易公开课的例子为例

之前koa博客地址：https://blog.csdn.net/frank_come/article/details/80805032

项目地址：https://github.com/WeForStudy/Lottery-node

![image-20180809143735025](/Users/wuyunhe/Desktop/image-2.png)

红圈部分是要扒取的数据

首先我们来看一下项目目录

![image-20180809142858655](/Users/wuyunhe/Desktop/image-1.png)

我们是在之前koa项目的基础上添加了爬虫的功能

新添的文件

- reptile.js

我们来看一下

```js
const ReptileService = require('./services/reptile')
const app = require('./index')
const puppeteer = require('puppeteer');
(async() => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  const url = "https://open.163.com/"
  await page.goto(url);
  const courses = await page.evaluate(() => {
    const coursesList = Array.from(
      document.querySelectorAll('.j-hotlist .item')
    )
    const getText = (e, selector) => {
      return e.querySelector(selector) && e.querySelector(selector).innerText
    }
    const data = coursesList.map(e => {
      const obj = {
        num: getText(e, '.icon'),
        text: getText(e, 'span'),
      }
      return obj
    })
    return data
  })
  // 拿到数据以后插入数据库
  await courses.map(async item => {
    const cV = JSON.stringify(item)
    const res = await ReptileService.add({
      url,
      contentValue: cV,
      type: 1000, // 代表是内容
    }, true)
    if (res) {
      console.log(`调用ReptileService添加对象成功,对象值为:${cV}`)
    } else {
      console.log(`调用ReptileService添加对象失败,原因为:${res}`)
    }
  })
  await page.close()
  await browser.close()
  app()
})();
```



首先我们打开网页以后，打开谷歌开发者工具（F12或者鼠标右键选择），分析一下数据所在的div结构，以本demo为例

![image-20180809144320761](/Users/wuyunhe/Desktop/image-3.png)

我们可以清晰地看到想要的数据是在<div>.j-hotlist里的<a>标签.item里的<i>和<span>

我们来看一下代码

在打开网页后，我们用puppeteer内置的evaluate方法进入浏览器环境，然后获得到对应的HTMLElement节点

```js
const courses = await page.evaluate(() => {
    // evaluate方法是在浏览器环境下运行的一个匿名函数
    // 可以获取浏览器环境下等价的Bom操作
    // Document、 Window etc.
    // 注意的是，内部是一个隔离的环境，可以通过第二个参数把参数传过来evaluate(func, params)
    // 分析对应的数据结构
    const coursesList = Array.from(
      document.querySelectorAll('.j-hotlist .item')
    )
    // 获取相应元素内部子元素的innerText
    const getText = (e, selector) => {
      return e.querySelector(selector) && e.querySelector(selector).innerText
    }
    // 组合数据
    const data = coursesList.map(e => {
      const obj = {
        num: getText(e, '.icon'),
        text: getText(e, 'span'),
      }
      return obj
    })
    // 返回
    return data
  })
```

这是我们拿到的数据。

```json
[ { num: '01', text: '你真的了解消化不良吗？\t\t\t\t' },
  { num: '02', text: '孩子爱流眼泪?或是青光眼!\t\t\t\t' },
  { num: '03', text: '艾滋病已变可控的慢性病？\t\t\t\t' },
  { num: '04', text: '查出泌尿结石 医生了赐8个字' },
  { num: '05', text: '如何判断是否感染肺结核？\t\t\t\t' },
  { num: '06', text: '跑步竟然能治疗泌尿结石？' },
  { num: '07', text: '慢性胃炎当心变胃癌！\t\t\t\t' },
  { num: '08', text: '尿不成直线是前列腺有问题？' },
  { num: '09', text: '幽门螺杆菌该如何检测？\t\t\t\t' },
  { num: '10', text: '大活人会被尿给憋死吗？' } ]
```

接下来就是对数据库的操作了

我们调用Services层

```js
 // 拿到数据以后插入数据库
  await courses.map(async item => {
    const cV = JSON.stringify(item)
    // 调用service的添加
    const res = await ReptileService.add({
      url,
      contentValue: cV,
      type: 1000, // 代表是内容
    }, true)
    if (res) {
      console.log(`调用ReptileService添加对象成功,对象值为:${cV}`)
    } else {
      console.log(`调用ReptileService添加对象失败,原因为:${res}`)
    }
  })
```

其中url代表来源，contentValue代表我们扒取到的内容，type代表是文字还是图片，

当然这个设计很简单，也只是为了让爬虫的功能和数据库贯穿起来，我们就不纠结这个数据库的设计了

接下来我们来看一下services层的内容

```js
const controller = require('../controller/reptile')
const pojo = require('../helper/pojo')
const model = require('./model')
const { success, failed, filterUnderLine }  = pojo
const m  = model([
  'list',
], 'reptile')
/**
 * @description 重写add，为了给爬虫新添一些逻辑
 * @param {*} ctx   如果是node环境调用就是params
 * @param {*} isNode 如果是node环境调用（非api）
 */
const add = async (ctx, isNode = false) => {
  let res;
  try {
    let val;
    if (isNode) {
      val = ctx
    } else {
      val = ctx.request.body
    }
    // 调用controller的add方法
    await controller.add(val).then(result => {
      if (isNode) {
        // node调取返回影响的行数
        res = result.affectedRows
        return
      }
      if(result.length === 0 || result === null || result === undefined)  
        res = failed('操作失败')
      else 
        res = success(filterUnderLine(result[0]))
    })
  } catch(err) {
    res = failed(err)
  }
  if (isNode) {
    // node调取返回bool
    return res >= 1
  } else {
    ctx.body = res
  }
}
module.exports = {
  ...m,
  add,
}
```



其中add方法分为两种环境调用，node调用和正常api调用



接下来是controller

```js
// 在lib下封装好的mysql数据库连接池
const pool = require('../lib/mysql')
// STATUS是定义的枚举对象
const { STATUS } = require('../enum')
// 封装好的数据库连接池对象
const { query } = pool
// 新添管理员
const add = (val) => {
  const { url, contentValue, type } = val
  const values = Object.values(val)
  const _sql = 'insert into reptile(url,content_value,type,create_time,status) values(?,?,?,now(),?);'
  return query( _sql, [ url, contentValue, type, STATUS.NORMAL])
}
const list = () => {
  const _sql = 'select * from reptile where status =? ;'
  return query( _sql, [STATUS.NORMAL])
}

module.exports = {
  add,
  list,
}
```



list为正常的api,

到这里，我们就完成了对数据库的操作了

`yarn run craw`

在进行爬虫的同时打开了koa的数据服务

项目地址：https://github.com/WeForStudy/puppeteer-reptile