# Intermediate Usage

本文档介绍了如何构建一个更复杂的 Flask-RESTful 应用程序，并概述了在构建真实世界 Flask-RESTful 驱动 API 时应遵循的最佳实践。

## 项目结构

有许多不同的方法来组织您的 Flask-RESTful 应用程序，但这里我们将描述一种在大型应用程序中效果很好且保持良好组织水平的方法。

基本思想是将您的应用程序分为三个主要部分：路由、资源和任何通用基础设施。

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

common 目录可能只包含一个存储库函数集，以满足您的应用程序在整个应用程序中的需求。它还可以包含，例如，您的资源需要完成工作的任何自定义输入/输出类型。

在资源文件中，您只需要您的资源对象。所以 `foo.py` 可能看起来像这样：
```python
from flask_restful import Resource

class Foo(Resource):
    def get(self):
        pass
    def post(self):
        pass
```

