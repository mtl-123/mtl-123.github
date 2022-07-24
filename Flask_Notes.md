
# Flask 开发

## 路由

### 路由的第二种写法

> 路由除了装饰器的写法，还有其他写法吗？

```python
from flask import Flask

app = Flask(__name__)

# 第一种路由使用方式
@app.route('/')
def index():
    return "Hello World!"

def home():
    return "HOME"

# 路由的第二种方式，第一个参数是url路径，view_func参数对应的是视图函数
# app.add_utl_rule('/home', view_func=home)
# 
# 完整写法 url路径 别名  视图函数
app.add_url_rule('/home', 'home', home)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

### 反向生成url

```python
from flask import Flask, url_for

app = Flask(__name__)

# 第一种路由使用方式
@app.route('/')
def  index():
    return "Hello World!"

def home():
    print(url_for('home'))  # /home
    print(url_for('index')) # /
    return "Home"
# 理由额第二种方式， 第一个参数是url路径，view_func参数对应的是视图函数
# app.add_url_rble('/home', view_func=home)

# 完整写法  url路径  别名   视图函数
app.add_url_rule('/home', 'home', home)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

如果是使用装饰器的情况下，路由的别名就是函数名！！！

### 路由别名的坑

```python
from flask import Flask, request

app = Flask(__name__)


def decorator(fn):
    def inner(*args, **kwargs):
        print('来了')
        res = fn(*args, **kwargs)
        print('走了')
        return res
    return inner


@app.route('/')
@decorator
def index():
    print(request.endpoint)
    return "Hello world"


@app.route('/home')
@decorator
def home():
    return "Home"


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')     
```

项目运行会报错
> 报错：AssertionError: View function mapping is overwriting an existing endpoint function: index

原因：
被装饰器装饰的这个函数，它的 endpoint就会变成实际执行的这个函数，最终就会变成inner这个函数名
如果两个路由的别名是一样的，就会报错

验证：

```python
app.add_url_rule('/', 'index', index)
app.add_url_rule('/home', 'index', home)
```

## 视图

### 请求对象 request

1. 需要导入，局部对象，只能在视图函数中使用
2. 当content_type为application/json的时候，request.json就是传递来的json数据，并且会反序列化json为dict，如果不是，那么调用这个属性就会直接截断
3. request.data 只有当content_type为application的时候才有值，其值为bytes

```python
from flask import Flask, request

app = Flask(__name__)


@app.route('/', methods=['GET', 'POST'])
def index():
    # request是请求对象 局部对象 只能在视图中使用
    print(request.endpoint)  # 别名
    print(request.path)  # 访问的路径
    print(request.full_path)    # 访问完整的路径
    # print(request.headers)  # 请求头
    print("查询参数:", request.query_string)  # 查询参数 b'name=%E5%BC%A0%E4%B8%89'
    print(request.json)  # 针对content_type为application/json的请求 {"user": "xxx"}
    print(request.content_type)
    print(request.data)  # 原始请求体
    print(request.form)     # 表单 会解析x-www-form-urlencoded和form-data的请求体
    print(request.method)   # 指定请求方式如['GET', 'POST']

    return "Hello World"


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

## FBV和CBV

[Flask官网views 视图](https://flask.palletsprojects.com/en/2.1.x/tutorial/views/#)

### 第一种类视图
>
>1. 直接继承自 views.View
>2. 注意：必须要重写 dispatch_request 这个方法
>
```python
from flask import Flask, views, request

app = Flask(__name__)

# 第一种类视图
class IndexView(views.View):
    methods = ["GET", "POST"]

    def dispatch_request(self):
        if request.method == 'GET':
            return self.get()
        return self.post(2)

    def get(self):
        return "GET"

    def post(self, num):
        print(num)
        return "POST"


app.add_url_rule('/', view_func=IndexView.as_view(name='index'))

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

### 第二种类视图

```python
from flask import Flask, views, request

app = Flask(__name__)
# 第二种类视图
class HomeView(views.MethodView):
    methods = ["GET", "POST"]

    def get(self):
        return "GET"

    def POST(self):
        return "POST"


app.add_url_rule('/home/', view_func=HomeView.as_view(name='home'))

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```
## 响应
>1. flask里面可以直接返回字符串
>2. 返回字典 --> 会自动变为json  响应头: application/json
```python
from flask import Flask, jsonify, make_response

app = Flask(__name__)


@app.route('/')
def index():
    # 直接返回字符串
    return "index"
    # 返回字段
    return '{"msg", "您好"}'
    return jsonify({"jsonify", "您好"})

    res = make_response(jsonify({"msg", "您好"}))  # 响应对象
    res.headers['xxx'] = 'xxx'  # 设置响应头
    res.set_cookie('xxx', 'yyy', 10)  # 设置cookie
    return res


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')

```
