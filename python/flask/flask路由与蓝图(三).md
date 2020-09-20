# flask路由与蓝图

## 路由

```python
@app.route("/itcast")
def view_func():
    return "hello world"
```

### 1 查询路由信息

- 命令行方式

  ```shell
  flask routes
  ```

  ```shell
  Endpoint  Methods  Rule
  --------  -------  -----------------------
  index     GET      /
  static    GET      /static/
  ```

- 在程序中获取

  在应用中的url_map属性中保存着整个Flask应用的路由映射信息，可以通过读取这个属性获取路由信息

  ```python
  print(app.url_map)
  ```

  如果想在程序中遍历路由信息，可以采用如下方式

  ```python
  for rule in app.url_map.iter_rules():
      print('name={} path={}'.format(rule.endpoint, rule.rule))
  ```

#### 需求

通过访问`/`地址，以json的方式返回应用内的所有路由信息

#### 实现

```python
@app.route('/')
def route_map():
    """
    主视图，返回所有视图网址
    """
    rules_iterator = app.url_map.iter_rules()
    return json.dumps({rule.endpoint: rule.rule for rule in rules_iterator})
```

### 2 指定请求方式

在 Flask 中，定义路由其默认的请求方式为：

- GET
- OPTIONS(自带)
- HEAD(自带)

利用`methods`参数可以自己指定一个接口的请求方式

```python
@app.route("/itcast1", methods=["POST"])
def view_func_1():
    return "hello world 1"

@app.route("/itcast2", methods=["GET", "POST"])
def view_func_2():
    return "hello world 2"
```

## 蓝图

### 需求

在一个Flask 应用项目中，如果业务视图过多，可否将以某种方式划分出的业务单元单独维护，将每个单元用到的视图、静态文件、模板文件等独立分开？

例如从业务角度上，可将整个应用划分为用户模块单元、商品模块单元、订单模块单元，如何分别开发这些不同单元，并最终整合到一个项目应用中？

在Django中这种需求是如何实现的？

### 蓝图

在Flask中，使用蓝图Blueprint来分模块组织管理。

蓝图实际可以理解为是一个存储一组视图方法的容器对象，其具有如下特点：

- 一个应用可以具有多个Blueprint
- 可以将一个Blueprint注册到任何一个未使用的URL下比如 “/user”、“/goods”
- Blueprint可以单独具有自己的模板、静态文件或者其它的通用操作方法，它并不是必须要实现应用的视图和函数的
- 在一个应用初始化时，就应该要注册需要使用的Blueprint

但是一个Blueprint并不是一个完整的应用，它不能独立于应用运行，而必须要注册到某一个应用中。

### 使用方式

使用蓝图可以分为三个步骤

1. 创建一个蓝图对象

   ```python
    user_bp=Blueprint('user',__name__)
   ```

2. 在这个蓝图对象上进行操作,注册路由,指定静态文件夹,注册模版过滤器

   ```python
    @user_bp.route('/')
    def user_profile():
        return 'user_profile'
   ```

3. 在应用对象上注册这个蓝图对象

   ```python
    app.register_blueprint(user_bp)
   ```

#### 单文件蓝图

可以将创建蓝图对象与定义视图放到一个文件中 。

#### 目录（包）蓝图

对于一个打算包含多个文件的蓝图，通常将创建蓝图对象放到Python包的`__init__.py`文件中

```shell
--------- project # 工程目录
  |------ main.py # 启动文件
  |------ user  #用户蓝图
  |  |--- __init__.py  # 此处创建蓝图对象
  |  |--- passport.py  
  |  |--- profile.py
  |  |--- ...
  |
  |------ goods # 商品蓝图
  |  |--- __init__.py
  |  |--- ...
  |...
```

### 扩展用法

#### 1 指定蓝图的url前缀

在应用中注册蓝图时使用`url_prefix`参数指定

```python
app.register_blueprint(user_bp, url_prefix='/user')
app.register_blueprint(goods_bp, url_prefix='/goods')
```

#### 2 蓝图内部静态文件

和应用对象不同，蓝图对象创建时不会默认注册静态目录的路由。需要我们在 创建时指定 static_folder 参数。

下面的示例将蓝图所在目录下的static_admin目录设置为静态目录

```python
admin = Blueprint("admin",__name__,static_folder='static_admin')
app.register_blueprint(admin,url_prefix='/admin')
```

现在就可以使用`/admin/static_admin/<filename>`访问`static_admin`目录下的静态文件了。

也可通过`static_url_path`改变访问路径

```python
admin = Blueprint("admin",__name__,static_folder='static_admin',static_url_path='/lib')
app.register_blueprint(admin,url_prefix='/admin')
```

#### 3 蓝图内部模板目录

蓝图对象默认的模板目录为系统的模版目录，可以在创建蓝图对象时使用 template_folder 关键字参数设置模板目录

```python
admin = Blueprint('admin',__name__,template_folder='my_templates')
```