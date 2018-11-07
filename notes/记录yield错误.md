### 记录yield错误

> 今天在项目里面写一个函数时报错，所以记录下来

yield是generator语法糖的提交

我们来看一个例子

```

* function abc() {
    yield 1
}

abc.next()// 1

```

正常情况下，使用在函数体里使用多个yield是没有问题的

```
* function abc() {
    yield 1
    yield 2
}

abc.next()// 1
abc.next()// 2
```

但是当你在循环体里面使用yield的时候就出错了

```
* function abc() {
    yield 1
    while(true) {
        yield 2 // 报错
    }
}

```

所以在循环体里面是不可以使用yield的

同理在async/await