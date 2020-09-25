# MySQL必知必会系列

**整理作者阅读该书的知识点**

## 第５章 排序检索数据

### 5.1 排序数据

一般我们用select查询语句的时候,显示的数据一般将以它在底层表中出现的顺序显示。这可以是数据最初添加到表中的顺序。但是，如果数据后来进行过更新或删除，则此顺序将会受到MySQL重用回收存储空间的影响。因此，如果不明确控制的话，不能（也不应该）依赖该排序顺序。关系数据库设计理论认为，如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有意义。

为了明确地排序用SELECT语句检索出的数据，可使用ORDER BY子句。
ORDER BY子句取一个或多个列的名字，据此对输出进行排序。请看下面
的例子：

+ 单字段排序

```sql
select prod_name from products order by prod_name;

注意:
通过非选择列进行排序
通常，ORDER BY子句中使用的列将
是为显示所选择的列。但是，实际上并不一定要这样，用非
检索的列排序数据是完全合法的。
```

+ 多字段排序

```sql
select prod_name,prod_id from products order by prod_id,prod_name;

# 分析
先以prod_id进行排序,当有两个相同的prod_id,则通过prod_name进行比较
```

+ 以降序方式排序

```sql
select prod_id,prod_name from products order by prod_id desc,prod_name;
# desc表示降序
# 分析
以prod_id字段进行降序,prod_name升序
当不写desc时,默认升序
```

+ 配合limit

```sql
select prod_price from products order by prod_price desc limit 1;
# limit 必须在order by 后面
# 分析
返回表中价格最高的检索行
```

## 第６章 过滤数据

### 6.1 使用WHERE子句

一般来说,我们并不需要一张表中的所有数据,这时我们就可以通过where子句来筛选我们

想要的数据.

在SELECT语句中，数据根据WHERE子句中指定的搜索条件进行过滤。
WHERE子句在表名（FROM子句）之后给出

```sql
select prod_price from products where prod_price = 2.50;

# 分析 
返回prod_price = 2.50所在的行
# 注意
在使用order by 和limit的时候
order by 要在where后面
```

### 6.2 WHERE子句操作符

```sql
=   等于
!=  不等于
<   小于
>   大于
>=  大于等于
<=   小于等于
between min and max 在指定的两个值之间(该区间包含两端的值)
is null 匹配该列中值为null的,该null不同于数字0,空字符和包含空格的字符
```

