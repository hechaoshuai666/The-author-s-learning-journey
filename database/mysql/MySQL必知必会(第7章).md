# MySQL必知必会系列

**整理作者阅读该书的知识点**

## 第 7 章 数据过滤

### 7.1 组合WHERE子句

为了进行更强的过滤控制，MySQL允许给出多个**WHERE**子句。这些子
句可以两种方式使用：以**AND**子句的方式或**OR**子句的方式使用。

操作符（operator） 用来联结或改变WHERE子句中的子句的关键
字。也称为逻辑操作符（logical operator）。

#### 7.1.1 AND操作符

```sql
输入:SELECT prod_id,prod_price,prod_name FROM products WHERE vend_id = 1003 and prod_price <= 10;

分析:AND指示DBMS只返回满足所有给
定条件的行。如果某个产品由供应商1003制造，但它的价格高于10美元，
则不检索它。类似，如果产品价格小于10美元

注意:
上述例子中使用了只包含一个关键字AND的语句，把两个过滤条件组
合在一起。还可以添加多个过滤条件，每添加一条就要使用一个AND。
```

#### 7.1.2 OR操作符

OR操作符与AND操作符不同，它指示MySQL检索匹配任一条件的行。

```sql
select prod_id,prod_price,prod_name from products where vend_id = 1003 or prod_price = 10;
```

#### 7.1.3 计算次序

WHERE可包含任意数目的AND和OR操作符。允许两者结合以进行复杂
和高级的过滤。

+ **注意**
  + 当包含and 和 or在一起时,and的优先级要大于or

### 7.2 IN操作符

IN取合法值的由逗号分隔的清单，全都括在圆括号中

语法: where field in (condition,...)

细心的朋友可能发现了,其实in和or本质上没啥区别,但in有以下的优势

+ 在使用长的合法选项清单时，IN操作符的语法更清楚且更直观。
+ IN操作符一般比OR操作符清单执行更快。

### 7.3 NOT操作符

WHERE子句中的NOT操作符有且只有一个功能，那就是否定它之后所
跟的任何条件。

语法: where field not in (condition,...)