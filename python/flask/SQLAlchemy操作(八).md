# SQLAlchemy操作

## 1 新增

```python
user = User(mobile='15612345678', name='itcast')
db.session.add(user)
db.session.commit()
profile = Profile(id=user.id)
db.session.add(profile)
db.session.commit()
```

对于批量添加也可使用如下语法

```python
db.session.add_all([user1, user2, user3])
db.session.commit()
```

## 2 查询

#### all()

查询所有，返回列表

```python
User.query.all()
```

#### first()

查询第一个，返回对象

```python
User.query.first()
```

#### get()

根据主键ID获取对象，若主键不存在返回None

```python
User.query.get(2)
```

#### 另一种查询方式

```python
db.session.query(User).all()
db.session.query(User).first()
db.session.query(User).get(2)
```

#### filter_by

进行过虑

```python
User.query.filter_by(mobile='13911111111').first()
User.query.filter_by(mobile='13911111111', id=1).first()  # and关系
```

#### filter

进行过虑

```python
User.query.filter(User.mobile=='13911111111').first()
```

#### 逻辑或

```python
from sqlalchemy import or_
User.query.filter(or_(User.mobile=='13911111111', User.name.endswith('号'))).all()
```

#### 逻辑与

```python
from sqlalchemy import and_
User.query.filter(and_(User.name != '13911111111', User.mobile.startswith('185'))).all()
```

#### 逻辑非

```python
from sqlalchemy import not_
User.query.filter(not_(User.mobile == '13911111111')).all()
```

#### offset

偏移，起始位置

```python
User.query.offset(2).all()
```

#### limit

获取限制数据

```python
User.query.limit(3).all()
```

#### order_by

排序

```python
User.query.order_by(User.id).all()  # 正序
User.query.order_by(User.id.desc()).all()  # 倒序
```

#### 复合查询

```python
User.query.filter(User.name.startswith('13')).order_by(User.id.desc()).offset(2).limit(5).all()
query = User.query.filter(User.name.startswith('13'))
query = query.order_by(User.id.desc())
query = query.offset(2).limit(5)
ret = query.all()
```

#### 优化查询

```python
user = User.query.filter_by(id=1).first()  # 查询所有字段
select user_id, mobile......

select * from   # 程序不要使用
select user_id, mobile,.... # 查询指定字段

from sqlalchemy.orm import load_only
User.query.options(load_only(User.name, User.mobile)).filter_by(id=1).first() # 查询特定字段
```

#### 聚合查询

```python
from sqlalchemy import func

db.session.query(Relation.user_id, func.count(Relation.target_user_id)).filter(Relation.relation == Relation.RELATION.FOLLOW).group_by(Relation.user_id).all()
```

#### 关联查询

##### 1. 使用ForeignKey

```python
class User(db.Model):
    ...
    profile = db.relationship('UserProfile', uselist=False)
    followings = db.relationship('Relation')

class UserProfile(db.Model):
    id = db.Column('user_id', db.Integer, db.ForeignKey('user_basic.user_id'), primary_key=True,  doc='用户ID')
    ...

class Relation(db.Model):
    user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), doc='用户ID')
    ...

# 测试   
user = User.query.get(1)
user.profile.gender
user.followings
```

##### 2. 使用primaryjoin

```python
class User(db.Model):
    ...

    profile = db.relationship('UserProfile', primaryjoin='User.id==foreign(UserProfile.id)', uselist=False)
    followings = db.relationship('Relation', primaryjoin='User.id==foreign(Relation.user_id)')

# 测试
user = User.query.get(1)
user.profile.gender
user.followings
```

##### 3. 指定字段关联查询

```python
class Relation(db.Model):
    ...
    target_user = db.relationship('User', primaryjoin='Relation.target_user_id==foreign(User.id)', uselist=False)

from sqlalchemy.orm import load_only, contains_eager

Relation.query.join(Relation.target_user).options(load_only(Relation.target_user_id), contains_eager(Relation.target_user).load_only(User.name)).all()
```

## 3 更新

- 方式一

  ```python
    user = User.query.get(1)
    user.name = 'Python'
    db.session.add(user)
    db.session.commit()
  ```

- 方式二

  ```python
    User.query.filter_by(id=1).update({'name':'python'})
    db.session.commit()
  ```

## 4 删除

- 方式一

  ```python
    user = User.query.order_by(User.id.desc()).first()
    db.session.delete(user)
    db.session.commit()
  ```

- 方式二

  ```python
    User.query.filter(User.mobile='18512345678').delete()
    db.session.commit()
  ```

## 5 事务

```python
environ = {'wsgi.version':(1,0), 'wsgi.input': '', 'REQUEST_METHOD': 'GET', 'PATH_INFO': '/', 'SERVER_NAME': 'itcast server', 'wsgi.url_scheme': 'http', 'SERVER_PORT': '80'}

with app.request_context(environ):
    try:
        user = User(mobile='18911111111', name='itheima')
        db.session.add(user)
        db.session.flush() # 将db.session记录的sql传到数据库中执行
        profile = UserProfile(id=user.id)
        db.session.add(profile)
        db.session.commit()
    except:
        db.session.rollback()
```