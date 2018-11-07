#### PhantomJs 入门小demo

> 为啥要用PhantomJs呢，而不是superagent + cheerio？后者只可以爬取一些静态网页是非常好用的，但是遇到了动态JS加载数据的网页就束手无策了。所以这里选取了phantomJs作为爬虫的工具（又称无头浏览器）

- 官网下载phantomJs的exe程序

- 配置phantomJs环境变量

  - mac配置

    ##### 我用的是Iterm2 + Oh my zash 所以我配置的是.zshrc文件

    ```
    alias phantomjs='/Applications/phantomjs/bin/phantomjs'
    ```

- 编写Js文件

  例子

  ```
  var page = require('webpage').create();
  phantom.outputEncoding="utf-8";//指定编码方式
  page.open("http://news.163.com/", function(status) {
  if ( status === "success" ) {
  	// 加载成功
  	console.log(page.content) // 输出全部网页内容（动态加载数据以后的网页）
  } else {
  console.log("网页加载失败");
  }
  phantom.exit(0);//退出系统
  });
  ```

  

