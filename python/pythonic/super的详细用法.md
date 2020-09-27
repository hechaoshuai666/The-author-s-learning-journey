# super的详细用法

让我们看看super的官方定义

https://docs.python.org/3.6/library/functions.html?highlight=super#super

根据官方定义的super用法

super([type[, object or type]])

返回一个代理对象，该对象将方法调用委托给类的父类或兄弟类。这对于访问类中已重写的继承方法非常有用。搜索顺序与getattr()使用的搜索顺序相同，只是类型本身被跳过。

看到解释就可以很明显的知道,其实super并不是得到了父类,而是通过一个代理对象,来访问父类的方法

我们看看super的用法

那我们再看看super的使用方法：

```
super() -> same as super(__class__, <first argument>)
super(type) -> unbound super object
super(type, obj) -> bound super object; requires isinstance(obj, type)
super(type, type2) -> bound super object; requires issubclass(type2, type)
```

super至少需要一个参数，并且类型需要是类。

不传参数的会报错。只传一个参数的话是一个不绑定的对象，不绑定的话也就没什么用了。

接下来我们来研究下其他几种的用法

+ super()

  如果是在类中调用super方法,则默认是调用super(cls,cls()),根据mro算法,在多继承方面,并不是深度的

  调用父类方法,而是广度优先

  super()是根据第二个参数来计算MRO，根据顺序查找第一个参数类后的方法。

+ super(type,obj),同上

+ super(type,type2),第二个参数接受值为第一个参数的子类,那么跟super(type,obj)有何差别呢,

  super()第二个参数是类，得到的方法是函数，使用时要传self参数。第二个参数是对象，得到的是绑定方法，不需要再传self参数。

  