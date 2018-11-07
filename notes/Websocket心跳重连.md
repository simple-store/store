Websocket心跳重连

> 最近的工作业务上有用到关于websocket的相关知识，本来打算用socket.io去完成，但是最后还是采用了自带的websocket的方式完成需求

- 新建websocket

  ```js
   this.websocket = new WebSocket('yoursocketurl')
  ```

- 设置websocket传输类型(默认为text/json，这里设置的是二进制)

  ```js
    this.websocket.binaryType = "arraybuffer";    
  ```

- 监听open事件

  ```js
   this.websocket.addEventListener('open', (event) => {
   })
  ```

- 监听message事件

  > websocket最重要的业务逻辑就在这里处理，每当收到服务器发送过来的消息，此周期便会触发一次

  ```js
  this.websocket.addEventListener('message', event => {
      // 这里写你的业务逻辑代码
  })
  ```

- 监听close事件

  ```js
  this.websocket.addEventListener('close', (event) => {
  })
  ```



##### 现在想象一下有这么一个场景，如果网络状态不佳，web端和服务器的websocket突然断了一次，整个逻辑都会被毁，所以我们采用心跳重连的方式去重新连接

- 修改close

  ```js
   this.websocket.addEventListener('close', () => {
     this.addWebsocket() // 这里重新连接websocket
    })
  ```

  - addWebsocket代码、

    > 其实就是把整个代码放进一个方法，调用一次

    ```js
    const addWebsocket = e => {
       this.websocket = new WebSocket('yoursocketurl')
       this.websocket.binaryType = "arraybuffer"
       this.websocket.addEventListener('open', (event) => {
       })
       this.websocket.addEventListener('message', event => {
         // 这里写你的业务逻辑代码
       })
       this.websocket.addEventListener('close', () => {
         this.addWebsocket() // 这里重新连接websocket
       })
    }
    ```

- 到这里就可以肯定能够心跳重连了

  > 但是还需要考虑一个事情，就是当你的网络状态不佳时，它会尝试一直连接到所以我们需要设置一个limitTime

  ```js
   
   this.websocket.addEventListener('close', () => {
     if (this.recTimes < 59) {
       this.recTimes =  this.recTimes +1
       this.addWebsocket(carId)
     }
   })
  ```

- 对应的open也要修改

  ```js
  this.websocket.addEventListener('open', (event) => {
    this.recTimes = 0
  })
  ```

  ##### 当然如果重新连接成功了，就没必要发送重连了，所以再添加一个字段

  ```js
  this.websocket.addEventListener('open', (event) => {
    this.isReconnect = true
     this.recTimes = 0
  })
  ```

  ##### 修改后的close

  ```js
  this.websocket.addEventListener('close', () => {
    if (this.isReconnect) {
      if (this.recTimes < 59) {
        this.recTimes =  this.recTimes +1
        this.addWebsocket(carId)
      }
    }
  })
  ```

  

##### 到这里就大功告成了！

