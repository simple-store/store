#### MySQL

> 今天用node里的koa2写后台，记录一些问题

#### SQL语句

- 创建表

```sql
create table table_name (
	column_name int primary key (主键) auto_increment,
    column_name varcar(50) not null
)
// 最后一个列的末尾不应该加上逗号
```

- 查询表

```sql
select * from table_name
```

- 新增

```sql
insert into table_name values(value1, value2, value3, ...)
```

- 修改

```
update table_name set column = ?(value)
```

- 删除
```
delete from table where condition
```



#### 数据类型

- varchar

  > 类String，需给长度：varchar(50)

- nvarchar

  > 同上

- date

  > 保留到日期： 2018-01-01

- datetime

  > 保留到秒： 2018-01-01 12:00:00

- long

  > 长整型

- int

  > 整型
  >
  > 自增的主键必须为此

- demical

  > 可计算至35位数
  >
  > 一般用来金额

