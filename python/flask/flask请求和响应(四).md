# flask请求和响应

## 处理请求

### 需求

在视图编写中需要读取客户端请求携带的数据时，如何才能正确的取出数据呢？

请求携带的数据可能出现在HTTP报文中的不同位置，需要使用不同的方法来获取参数。

### 1. URL路径参数（动态路由）

例如，有一个请求访问的接口地址为`/users/123`，其中123实际上为具体的请求参数，表明请求123号用户的信息。此时如何从url中提取出123的数据？

Flask不同于Django直接在定义路由时编写正则表达式的方式，而是采用转换器语法：

```python
@app.route('/users/<user_id>')
def user_info(user_id):
    print(type(user_id))
    return 'hello user {}'.format(user_id)
```

此处的`<>`即是一个转换器，默认为字符串类型，即将该位置的数据以字符串格式进行匹配、并以字符串为数据类型类型、 `user_id`为参数名传入视图。

#### Flask也提供其他类型的转换器

```python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
}
```

将上面的例子以整型匹配数据，可以如下使用：

```python
@app.route('/users/<int:user_id>')
def user_info(user_id):
    print(type(user_id))
    return 'hello user {}'.format(user_id)


@app.route('/users/<int(min=1):user_id>')
def user_info(user_id):
    print(type(user_id))
    return 'hello user {}'.format(user_id)
```

#### 自定义转换器

如果遇到需要匹配提取`/sms_codes/18512345678`中的手机号数据，Flask内置的转换器就无法满足需求，此时需要自定义转换器。

#### 定义方法

自定义转换器主要做3步

1. 创建转换器类，保存匹配时的正则表达式

   ```python
   from werkzeug.routing import BaseConverter
   
   class MobileConverter(BaseConverter):
       """
       手机号格式
       """
       regex = r'1[3-9]\d{9}'
   ```

   - 注意`regex`名字固定

2. 将自定义的转换器告知Flask应用

   ```python
   app = Flask(__name__)
   
   # 将自定义转换器添加到转换器字典中，并指定转换器使用时名字为: mobile
   app.url_map.converters['mobile'] = MobileConverter
   ```

3. 在使用转换器的地方定义使用

   ```python
   @app.route('/sms_codes/<mobile:mob_num>')
   def send_sms_code(mob_num):
       return 'send sms code to {}'.format(mob_num)
   ```

### 2. 其他参数

如果想要获取其他地方传递的参数，可以通过Flask提供的**request**对象来读取。

不同位置的参数都存放在request的不同属性中

| 属性    | 说明                           | 类型           |
| :------ | :----------------------------- | :------------- |
| data    | 记录请求的数据，并转换为字符串 | *              |
| form    | 记录请求中的表单数据           | MultiDict      |
| args    | 记录请求中的查询参数           | MultiDict      |
| cookies | 记录请求中的cookie信息         | Dict           |
| headers | 记录请求中的报文头             | EnvironHeaders |
| method  | 记录请求使用的HTTP方法         | GET/POST       |
| url     | 记录请求的URL地址              | string         |
| files   | 记录请求上传的文件             | *              |

例如 想要获取请求`/articles?channel_id=1`中`channel_id`的参数，可以按如下方式使用：

```python
from flask import request

@app.route('/articles')
def get_articles():
    channel_id = request.args.get('channel_id')
    return 'you wanna get articles of channel {}'.format(channel_id)
```

#### 上传图片

客户端上传图片到服务器，并保存到服务器中

```python
from flask import request

@app.route('/upload', methods=['POST'])
def upload_file():
    f = request.files['pic']
    # with open('./demo.png', 'wb') as new_file:
    #     new_file.write(f.read())
    f.save('./demo.png')
    return 'ok'
```

## 处理响应

### 需求

如何在不同的场景里返回不同的响应信息？

### 1 返回模板

使用`render_template`方法渲染模板并返回

例如，新建一个模板index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
我的模板html内容
<br/>{{ my_str }}
<br/>{{ my_int }}
</body>
</html>
```

后端视图

```python
from flask import render_template

@app.route('/')
def index():
    mstr = 'Hello 程序员'
    mint = 10
    return render_template('index.html', my_str=mstr, my_int=mint)
```

### 2 重定向

```python
from flask import redirect

@app.route('/demo2')
def demo2():
    return redirect('http://www.itheima.com')
```

### 3 返回JSON

```python
from flask import jsonify

@app.route('/demo3')
def demo3():
    json_dict = {
        "user_id": 10,
        "user_name": "laowang"
    }
    return jsonify(json_dict)
```

### 4 自定义状态码和响应头

#### 1） 元祖方式

可以返回一个元组，这样的元组必须是 **(response, status, headers)** 的形式，且至少包含一个元素。 status 值会覆盖状态代码， headers 可以是一个列表或字典，作为额外的消息标头值。

```python
@app.route('/demo4')
def demo4():
    # return '状态码为 666', 666
    # return '状态码为 666', 666, [('Itcast', 'Python')]
    return '状态码为 666', 666, {'Itcast': 'Python'}
```

#### 2) make_response方式

```python
@app.route('/demo5')
def demo5():
    resp = make_response('make response测试')
        resp.headers[“Itcast”] = “Python”
        resp.status = “404 not found”
    return resp
```

## Cookie与Session

### 1 Cookie

#### 设置

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/cookie')
def set_cookie():
    resp = make_response('set cookie ok')
    resp.set_cookie('username', 'itcast')
    return resp
```

设置有效期

```python
@app.route('/cookie')
def set_cookie():
    response = make_response('hello world')
    response.set_cookie('username', 'itheima', max_age=3600)
    return response
```

#### 读取

```python
from flask import request

@app.route('/get_cookie')
def get_cookie():
    resp = request.cookies.get('username')
    return resp
```

#### 删除

```python
from flask import request

@app.route('/delete_cookie')
def delete_cookie():
    response = make_response('hello world')
    response.delete_cookie('username')
    return response
```

### 2 Session

需要先设置SECRET_KEY

```python
class DefaultConfig(object):
    SECRET_KEY = 'fih9fh9eh9gh2'

app.config.from_object(DefaultConfig)

或者直接设置
app.secret_key='xihwidfw9efw'
```

#### 设置

```python
from flask import session

@app.route('/set_session')
def set_session():
    session['username'] = 'itcast'
    return 'set session ok'
```

#### 读取

```python
@app.route('/get_session')
def get_session():
    username = session.get('username')
    return 'get session username {}'.format(username)
```