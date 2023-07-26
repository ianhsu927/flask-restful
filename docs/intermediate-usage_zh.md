# Intermediate Usage

本文档介绍了如何构建一个更复杂的 `Flask-RESTful` 应用程序，并概述了在构建真实世界 `Flask-RESTful` 驱动 `API` 时应遵循的最佳实践。

## 项目结构

有许多不同的方法来组织您的 `Flask-RESTful` 应用程序，但这里我们将描述一种在大型应用程序中效果很好且保持良好组织水平的方法。

基本思想是将您的应用程序分为三个主要部分：路由(routes)、资源(resources)和任何通用基础设施(infrastructure)。

以下是一个示例目录结构：

```
myapi/
    __init__.py
    app.py          # 此文件包含您的应用程序和路由
    resources/
        __init__.py
        foo.py      # 包含 /Foo 逻辑
        bar.py      # 包含 /Bar 逻辑
    common/
        __init__.py
        util.py     # 只是一些通用基础设施
```

`common` 目录可能只包含一个存储库函数集，以满足您的应用程序在整个应用程序中的需求。它还可以包含，例如，您的资源需要完成工作的任何自定义输入/输出类型。

在资源文件中，您只需要您的资源对象。所以 `foo.py` 可能看起来像这样：
```python
from flask_restful import Resource

class Foo(Resource):
    def get(self):
        pass
    def post(self):
        pass
```

关键在于 app.py：

```python
from flask import Flask
from flask_restful import Api
from myapi.resources.foo import Foo
from myapi.resources.bar import Bar
from myapi.resources.baz import Baz

app = Flask(__name__)
api = Api(app)

api.add_resource(Foo, '/Foo', '/Foo/<string:id>')
api.add_resource(Bar, '/Bar', '/Bar/<string:id>')
api.add_resource(Baz, '/Baz', '/Baz/<string:id>')
```

如您所见，`app.py` 文件非常有价值，因为它包含了您的整个 API 的所有路由和资源的完整列表。您还会在该文件中使用 `config` 值 ([`before_request()`](https://flask.palletsprojects.com/en/2.3.x/api/#flask.Flask.before_request), [`after_request()`](https://flask.palletsprojects.com/en/2.3.x/api/#flask.Flask.after_request))。基本上，此文件配置您的整个 API。

`common` 目录中的内容只是您支持您的资源所需的基础设施。

## 使用 Blueprints

请参阅 Flask 文档中的 :ref:`blueprints` 部分了解蓝图是什么以及为什么您应该使用它们。以下是将 :class:`Api` 链接到 :class:`~flask.Blueprint` 的示例。

```python
from flask import Flask, Blueprint
from flask_restful import Api, Resource, url_for

app = Flask(__name__)
api_bp = Blueprint('api', __name__)
api = Api(api_bp)

class TodoItem(Resource):
    def get(self, id):
        return {'task': 'Say "Hello, World!"'}

api.add_resource(TodoItem, '/todos/<int:id>')
app.register_blueprint(api_bp)
```

> 调用 :meth:`Api.init_app` 在这里不是必需的，因为将蓝图注册到应用程序会自动处理路由的设置。

## 完整的参数解析示例

在 Flask-RESTful 文档的其他部分中，我们已经详细描述了如何使用 reqparse 示例。在这里，我们将设置一个具有多个输入参数的资源，以演示更大的选项数量。我们将定义一个名为“用户”的资源。

```python
from flask_restful import fields, marshal_with, reqparse, Resource

def email(email_str):
    """Return email_str if valid, raise an exception in other case."""
    if valid_email(email_str):
        return email_str
    else:
        raise ValueError('{} is not a valid email'.format(email_str))

post_parser = reqparse.RequestParser()
post_parser.add_argument(
    'username', dest='username',
    location='form', required=True,
    help='The user\'s username',
)
post_parser.add_argument(
    'email', dest='email',
    type=email, location='form',
    required=True, help='The user\'s email',
)
post_parser.add_argument(
    'user_priority', dest='user_priority',
    type=int, location='form',
    default=1, choices=range(5), help='The user\'s priority',
)

user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends'),
        'posts': fields.Url('user_posts'),
    }),
}

class User(Resource):

    @marshal_with(user_fields)
    def post(self):
        args = post_parser.parse_args()
        user = create_user(args.username, args.email, args.user_priority)
        return user

    @marshal_with(user_fields)
    def get(self, id):
        args = post_parser.parse_args()
        user = fetch_user(id)
        return user
```

