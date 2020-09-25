# MySQL必知必会系列

**整理作者阅读该书的知识点**

## 第三章 使用MySQL 

### 3.1 连接

为了连接到MySQL，需要以下信息：

+ 主机名（计算机名）——如果连接到本地MySQL服务器，为localhost
+ 端口（如果使用默认端口3306之外的端口）

### 3.2 选择数据库

在你最初连接到MySQL时，没有任何数据库打开供你使用。在你能执行任意数据库操作前，需要选择一个数据库。为此，可使用USE关键字。

```sql
关键字(key word) 作为MySQL语言组成部分的一个保留字。决
不要用关键字命名一个表或列。

例如，为了使用crashcourse数据库，应该输入以下内容：
输入:USE crashcourse
输出:Database changed

注:记住，必须先使用USE打开数据库，才能读取其中的数据。
```

### 3.3 了解数据库和表

+ 查看所有的数据库

  ```sql
  输入:SHOW DATABASES;
  分析:SHOW DATABASES;返回可用数据库的一个列表。包含在这个列
  表中的可能是MySQL内部使用的数据库（如例子中的mysql和
  information_schema）。当然，你自己的数据库列表可能看上去与这里的
  不一样。
  ```

+ 查看所有的表

  ```sql
  输入:SHOW TABLES;
  分析:SHOW TABLES;返回当前选择的数据库内可用表的列表。
  ```

+ 查看表结构

  ```sql
  输入:SHOW COLUMNS FROM 表名;
  简洁写法 desc 表名;
  
  查看详情结构
  show create table 表名
  show create database 数据库
  ```

+ 其他

  ```sql
  SHOW GRANTS，用来显示授予用户（所有用户或特定用户）的安
  全权限；
  ```

  