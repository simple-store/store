### RSA的node层应用

> 由于前几天有朋友问我RSA的使用方式，便花了一些时间去看了一下文档自己做了一下前端和node端
>
> 由来：由于表单提交的数据用f12或其他抓包工具可以看的一清二楚，所以前端在处理表单提交的时候会要求使用RSA/MD5等类似的加密工具加数据加密后再上传
>
> 用到的类库： node-rsa

- 对称性加密

  > 顾名思义，对称性加密就是两边的加密、解密方式用的是同一个秘方
  >
  > 你有两把相同的钥匙，你给了我一把钥匙和一个带锁的箱子，我把数据放进箱子交给你，你用另一把钥匙打开锁开启箱子拿到数据
  >
  > 两边的钥匙都是一样的，加密和解密用的是同一个key，称之为对称性加密

  - 优势

    ```
    后台只需要生成一个key，前台拿到钥匙直接加密数据
    ```

  - 劣势

    ```
    如果有不怀好意的人通过非正常手段拿到了前端保留的公钥便可以轻而易举地破解加密的数据，加密便没有了意义
    ```

    

- 非对称性加密

  > 加密、解密分别需要不同的秘方
  >
  > 你有两把钥匙，你给了我一把钥匙和一个带锁的箱子，我把数据放进箱子交给你，你用另一把钥匙打开锁开启箱子拿到数据
  >
  > 你给我的钥匙和你自己保留的钥匙不一致，开箱子必须用你那把钥匙，不同key,非对称性加密

  - 优势

    ```
    后台生成两个key，前台拿到钥匙后加密数据发给服务端，就算钥匙被其他人破解，也无法解密加密后的数据，保证了数据的绝密性
    ```

  - 劣势

    ```
    后台管理上需要多维护一个字段
    ```



#### 前端代码（非对称性加密）

> 实现逻辑
>
> - 先判断本地有没有缓存publicKey（有就直接用，没有就需要请求后台一次）
> - 调获取getKey接口
> - 拿到key后加密数据
> - 发送加密后的数据给服务端

我们来看代码

```js
import NodeRSA from 'node-rsa'
   
    ...
   
// Vue提交表单
submitForm(formName) {
      this.$refs[formName].validate(async valid => {
        if (valid) {
          // 判断是否有公钥存储，如果没有，则需要向服务端请求，一般存在sessionStorage/其他数据载体
          if (!this.publicKey) {
            // 向服务端发送请求，获得公钥
            await getPublicKey()
              .then(res => {
                this.publicKey = res
              })
              .catch(err => {
                this.$message.error(err.message || err || "网络错误");
            });
          }
          // 对数据进行加密，并获得加密后的数据
          const entryptData =  this.entryptData(this.models)
          
          // 拿到加密后的数据调用登录接口
          await RSALogin(
            entryptData,
          ).then(res => {
            this.$message({
              message: '登录成功',
              type: 'success'
            })
            // ... do something
          }).catch(err => {
             this.$message.error(err.message || err || '登录失败')
          })
        } else {
          this.$message.error("提交错误");
          return false;
        }
      });
    },
    entryptData(props) {
      // 生成RSA对象
      const RSAkey = new NodeRSA({ b: 512 })
      // 导入公钥
      RSAkey.importKey(this.publicKey)
      console.log('数据对象加密前：', props)
      const afterEncrypt = Object.keys(props).reduce((init, key) => {
        
        const item = props[key]
        console.log('字段加密前：', item)

        const entryptItem = RSAkey.encrypt(item, 'base64')
        console.log('字段加密后：', entryptItem)

        init[key] = entryptItem
        return init
      }, {})
      console.log('数据对象加密后：', afterEncrypt)
      return afterEncrypt
    },
```

到这里，我们就加密好了数据

```js
{
  account: "c1gYgdczUyXIJnNtqW4ytoAJ0S9kqAUeUnIZgmmyCZQ5qAFz62jG9ERUE0axkyEUVL5rn/PJPPVWGMLwoF25Ew==",
  password: "m0OOeuGl9HE1/SNjsLD39OF7N00pT/g25Wx4OfNS0BnnaFaDtNnXr69cXStsaugMkfwgvNA56GhDGo5j5YZEPQ=="
}
```



#### node实现

> 实现逻辑
>
> - 判断是否缓存了key
> - 发送给前端publicKey
> - 拿到加密后的数据
> - 解密
> - 调用正常的login接口

```js
const controller = require('../../controller/admin')
const pojo = require('../../helper/pojo')
const {
  success,
  failed,
} = pojo
const NodeRSA = require('node-rsa')
let myKey = {
  isInit: false,
};
const initKey = () => {
  const key = new NodeRSA({
    b: 512
  }); // 生成新的512位长度密钥

  // 如果被初始化过就是解密需要的步骤
  if (myKey.isInit) {
    key.importKey(myKey.privateKey)
    return key
  }

  // 新生成的对象，保留公钥和私钥
  const publicDer = key.exportKey('public'); // 公钥
  const privateDer = key.exportKey('private'); // 私钥
  myKey = {
    publicKey: publicDer,
    privateKey: privateDer,
    isInit: true,
  }
}

// 获取公钥的接口
const getKey = async ctx => {
  try {
    let res;
    // 判断是否存在RSA-key对象
    if (!myKey.isInit) {
      initKey() // 初始化key 对象
    }
    // 返回publicKey给前端
    res = success(myKey.publicKey)
    ctx.body = res
  } catch (err) {
    res = failed(err)
  }
}

// 登录接口
const login = async ctx => {
  let res;
  try {
    const val = ctx.request.body
    if (!myKey.isInit) {
      res = {
        retCode: 403,
        message: '密钥无法匹配，请重新获取密钥'
      }
    } else {
      const myKey = initKey()// 通过私钥获取RSA对象
      const data = Object.keys(val).reduce((init, key) => {
        const beforeDencrypt = val[key] // 解密前
        const item = myKey.decrypt(beforeDencrypt, 'utf8') // 解密后
        init[key] = item
        return init
      }, {})
      // 调用真正的登录接口
      // do something
    }
    ctx.body = res
  } catch (err) {
    res = failed(err)
  }

}
module.exports = {
  login,
  getKey,
}
```





完整的代码和项目在这里

