# Connexion
Connexion官方文档中文版，待优化

# 欢迎来到Connexion的文档

Connexion是一个基于Flask实现的一个框架，它基于以YAML格式描述的API的OpenAPI 2.0规范（以前称为Swagger Spec）自动处理HTTP请求。 Connexion允许您编写Swagger规范，然后将端点映射到您的Python函数。这使得它与其他基于Python代码生成规范的工具不同。您可以自由描述您的REST API，并尽可能详细地描述您的需求，然后Connexion保证它将按照您的设置运行工作。我们以这种方式建立Connexion以便：

* 简化开发流程
* 让API看起来更加清晰明确

# 快速开始

#### 运行条件

python2.7/3.4+

#### 安装

```
$ pip install connexion
```

#### 运行

把您的.yaml设置文件存放在项目根目录下的文件夹(比如swagger/)中，然后：

```
import connexion

app = connexion.FlaskApp(__name__, specification_dir="swagger/")
app.add_api("my_api.yaml")
app.run(port=8080)
```

#### 动态展示您的说明

Connexion使用Jinja2通过不定长参数来允许规范参数化。您可以为connexion.App构造函数中的应用程序或connexion中的每个特定API全局定义规范参数.App＃add_api方法：

```
app = connexion.FlaskApp(__name__, specification_dir='swagger/',
                    arguments={'global': 'global_value'})
app.add_api('my_api.yaml', arguments={'api_local': 'local_value'})
app.run(port=8080)
```

当在全局和API上提供值时，API值将优先。

#### Swagger UI 控制台

缺省情况下，API的Swagger UI在<font size=4 color="#FFB90F">base_path/ui/</font>中可用，其中<font size=4 color="#FFB90F">base\_path</font>是API的基本路径。

您可以在应用程序级别禁用Swagger UI：

```
app = connexion.FlaskApp(__name__, specification_dir='swagger/',
                    swagger_ui=False)
app.add_api('my_api.yaml')
```

您也可以在API级别禁用Swagger UI：

```
app = connexion.FlaskApp(__name__, specification_dir='swagger/')
app.add_api('my_api.yaml', swagger_ui=False)
```

#### 指定服务器

默认情况下，connexion使用默认的flask服务器，但您也可以使用Tornado或gevent作为HTTP服务器，以将服务器设置为Tornado或gevent：

```
import connexion

app = connexion.FlaskApp(__name__, port = 8080, specification_dir='swagger/', server='tornado')
```

# 命令行接口

为了方便，Connexion提供了一个命令行接口（CLI）。该接口旨在成为使用Connexion开发或测试OpenAPI规范的起点。

可用的命令是：

```
connexion run
```

所有命令都可以使用-h或-help来运行以列出更多信息。

#### 运行OpenAPI说明

Connexion CLI的子命令<font color="#FFB90F">run</font>使得即使在执行任何操作处理函数之前，也可以直接运行OpenAPI说明。这使您可以验证和检查API如何与Connexion配合使用。


要运行你的规范，在shell中执行：

```
$ connexion run your_api.yaml --stub --debug
```

该命令将告诉Connexion运行your_api.yaml规范文件，将stub操作（--stub）附加到API的不可用操作/函数以及调试模式（--debug）中。

这个命令的基本用法是：

```
$ connexion run [OPTIONS] SPEC_FILE [BASE_MODULE_PATH]
```

* SPEC_FILE：YAML格式的OpenAPI规范文件。
* BASE_MODULE_PATH（可选）：要从中导入API端点处理程序的文件系统路径。总之，你的Python代码保存的路径。


run命令有更多选项可用于完整列表运行：

```
$ connexion run --help
```

#### 运行模拟服务器

您可以运行一个简单的服务器，该服务器会在每个请求上返回示例响应，示例响应必须在OpenAPI说明的<font color="#FFB90F">examples</font>响应属性中定义。您的API说明文件不需要有任何<font color="#FFB90F">operationId</font>。

```
$ connexion run your_api.yaml --mock=all -v
```

# 路由

#### 路由映射视图函数


Connexion使用每个`Operation Object_`中的operationId来标识哪个Python函数应该处理每个URL。

显示路由：

```
paths:
  /hello_world:
    post:
      operationId: myapp.api.hello_world
```

如果您在规范POST请求中向<font color="#FFB90F">http://MYHOST/hello_world</font>提供了此路径，它将由myapp.api模块中的函数hello\_world处理。或者，您可以在操作定义中包含x-swagger-router-controller，使operationId是相对的：

```
paths:
  /hello_world:
    post:
      x-swagger-router-controller: myapp.api
      operationId: hello_world
```

#### 自动路由

为了定制这种行为，Connexion可以使用替代的<font color="#FFB90F">Resolvers</font>，例如<font color="#FFB90F">RestyResolver</font>。<font color="#FFB90F">RestyResolver</font>将根据您的规范中端点的路径和HTTP方法组成一个<font color="#FFB90F">operationId</font>：

```
from connexion.resolver import RestyResolver

app = connexion.FlaskApp(__name__)
app.add_api('swagger.yaml', resolver=RestyResolver('api'))
```

```
paths:
  /:
    get:
       # Implied operationId: api.get
  /foo:
    get:
       # Implied operationId: api.foo.search
    post:
       # Implied operationId: api.foo.post

  '/foo/{id}':
    get:
       # Implied operationId: api.foo.get
    put:
       # Implied operationId: api.foo.put
    copy:
       # Implied operationId: api.foo.copy
    delete:
       # Implied operationId: api.foo.delete
```

RestyResolver将优先于规范中遇到的任何operationId。它也将考虑x-swagger-router-controller。您可以导入并扩展connexion.resolver.Resolver以实现您自己的operationId（和函数）解析算法。

#### 参数名Sanitation

通过删除Python符号中不允许的字符来消除查询和表单参数的名称以及body参数的名称。即所有不是字母，数字或下划线的字符都将被删除，最后从前面删除字符，直到遇到字母或低于分数。举个例子

```
>>> re.sub('^[^a-zA-Z_]+', '', re.sub('[^0-9a-zA-Z_]', '', '$top'))
'top'
```

没有这个Sanitation就不可能实现[OData]（http://www.odata.org）API。

#### 参数变量转换器

Connexion支持Flask的int，float和path路由参数变量转换器。将路由参数的类型指定为整数或浮点数，字符串，并将其格式指定为使用这些转换器的<font color="#FFB90F">path</font>。例如：

```
paths:
  /greeting/{name}:
    # ...
    parameters:
      - name: name
        in: path
        required: true
        type: string
        format: path
```

上面代码将创建一个等效的Flask路由`/greeting/<path:name>`，允许请求在`name`url变量中包含正斜杠。

#### API Versioning和basePath

您也可以在API说明的顶层定义一个basePath。这对版本化的API很有用。eg:`http://MYHOST/1.0/hello_world`

```
basePath: /1.0

paths:
  /hello_world:
    post:
      operationId: myapp.api.hello_world
```

如果您不想在规范中包含基本路径，则可以在将API添加到应用程序时提供它:

```
app.add_api('my_api.yaml', base_path='/1.0')
```

#### Swagger JSON

Connexion在API的基本路径中以swagger.json的形式提供JSON格式的OpenAPI / Swagger说明。

您可以在应用程序级别禁用Swagger JSON:

```
app = connexion.FlaskApp(__name__, specification_dir='swagger/',
                    swagger_json=False)
app.add_api('my_api.yaml')
```

您也可以在API级别禁用Swagger JSON:

```
app = connexion.FlaskApp(__name__, specification_dir='swagger/')
app.add_api('my_api.yaml', swagger_json=False)
```

# 请求处理

Connexion验证传入的请求是否符合swagger规范中描述的模式。

如果请求参数包含在函数的签名中，则会将其作为关键字参数提供给处理函数，否则，可以从`connexion.request.json`访问主体参数，并且可以从`connexion.request.args`访问查询参数。

#### 请求验证

请求主体和参数都使用jsonschema根据规范进行验证。

如果请求不符合规范连接将返回400错误。

#### 自动参数处理

Connexion会自动将您的端点说明中定义的参数映射为您的Python视图的参数作为命名参数，并尽可能使用值转换。您需要做的就是视图参数的名称与端点参数定义一致。

例如，您有一个端点指定如下：

```
paths:
  /foo:
    get:
      operationId: api.foo_get
      parameters:
        - name: message
          description: Some message.
          in: query
          type: string
          required: true
```

相应的视图函数:

```
# api.py file

def foo_get(message):
    # do something
    return 'You send the message: {}'.format(message), 200
```

在这个例子中，Connexion会自动识别你的视图函数需要一个名为message的参数，并将端点参数消息的值分配给你的视图函数。

Connexion也将使用默认值（如果提供)

**注意：**
如果在端点上定义了一个不需要的参数，并且您的Python视图有一个非命名参数，那么当您在没有参数的情况下调用此端点时，您将获得缺少位置参数的异常。

#### 类型处理

只要有可能，Connexion就会尝试解析你的参数值，并将类型转换为相关的Python本地值。

<table>
    <tr>
        <th>Swagger Type</th>
        <th>Python Type</th>
    </tr>
    <tr>
        <th>integer</th>
        <th>int</th>
    </tr>
    <tr>
        <th>string</th>
        <th>str</th>
    </tr>
    <tr>
        <th>number</th>
        <th>float</th>
    </tr>
    <tr>
        <th>boolean</th>
        <th>bool</th>
    </tr>
    <tr>
        <th>array</th>
        <th>list</th>
    </tr>
    <tr>
        <th>object</th>
        <th>dict</th>
    </tr>
</table>

在Swagger定义中，如果使用数组类型，则可以定义它应该被识别的collectionFormat。 Connexion目前支持收集格式“pipes”和“csv”。默认格式是“csv”。

**提醒：**
有关collectionFormat的更多细节，请查看官方的Swagger / OpenAPI规范。

#### 参数验证

Connexion可以对查询和表单数据参数应用严格的参数验证。启用此项后，包含未在swagger规范中定义的参数的请求都将返回400错误。您可以在将API添加到应用程序时启用它

```
app.add_api('my_apy.yaml', strict_validation=True)
```

#### 空参数

有时你的API应该明确地接受可为空参数。然而，OpenAPI说明目前官方不支持服务此用例的方式，Connexion将x-nullable的扩展添加到参数定义中。它的用法是：

```
/countries/cities:
   parameters:
     - name: name
       in: query
       type: string
       x-nullable: true
       required: true
```

Connexion支持所有参数类型：body，query，formData和path。可为空的值是字符串null和None。

**注意：**
对于字符串“null”或“None”可以是有效值的敏感数据，请注意与可空参数的区别。

**提醒：**
只要OpenAPI / Swagger规范提供了支持空值的正式方法，该扩展就会被删除

#### 头参数

目前，头部参数不作为参数传递给处理函数。但是可以通过底层的connexion.request.headers对象来访问它们，该对象将flask.request.headers对象进行了别名化。

```
def index():
    page_number = connexion.request.headers['Page-Number']
```

#### 自定义验证器

缺省情况下，通过connexion.decorators.validation.RequestBodyValidator或connexion.decorators.validation.ParameterValidator，可以针对OpenAPI模式验证主体和参数内容，如果您想更改验证，则可以使用以下参数覆盖默认值：

```
validator_map = {
    'body': CustomRequestBodyValidator,
    'parameter': CustomParameterValidator
}
app = connexion.FlaskApp(__name__, ..., validator_map=validator_map)
```

请参阅examples / enforcedefaults中的自定义验证器示例。

# 响应处理

#### 响应序列化

如果端点返回一个Response对象，这个响应将按原样使用。

否则，默认情况下，如果规范定义端点只生成JSON，则connexion会自动序列化您的返回值，并在HTTP标头中设置正确的内容类型。

如果端点生成单个非JSON MIME类型，则Connexion将自动在HTTP头中设置正确的内容类型。

#### 定制JSON编码器

Connexion允许您在Flask应用程序实例json\_encoder（connexion.App：app）中自定义JSONEncoder类。如果您想重新使用Connexion的日期时间序列化，请从connexion.apps.flask_app.FlaskJSONEncoder继承您的自定义编码器。

#### 返回状态码

有两种方法可以返回特定的状态码。

一种方法是返回将保持不变的Response对象。

另一个将它作为响应中的第二个返回值返回。例如

```
def my_endpoint():
    return 'Not Found', 404
```

#### 返回标题

有两种方法可以从您的端点返回标题。

一种方法是返回将保持不变的Response对象。

另一个是返回一个带有标题值的字典作为响应中的第三个返回值：

例如：

```
def my_endpoint():
    return 'Not Found', 404, {'x-error': 'not found'}
```

#### 响应验证

虽然默认情况下，Connexion不会验证响应，但可以通过在添加API时选择这些响应来进行验证：

```
import connexion

app = connexion.FlaskApp(__name__, specification_dir='swagger/')
app.add_api('my_api.yaml', validate_responses=True)
app.run(port=8080)
```

这将使用jsonschema验证所有响应，并且在开发过程中特别有用。

#### 自定义验证器

默认情况下，响应正文内容通过connexion.decorators.response.ResponseValidator对OpenAPI模式进行验证，如果您想更改验证，则可以使用以下命令覆盖缺省类：

```
validator_map = {
    'response': CustomResponseValidator
}
app = connexion.FlaskApp(__name__, ..., validator_map=validator_map)
```

#### 错误处理

默认情况下，根据HTTP API的问题详细信息，连接错误消息是JSON序列化的

应用程序可以使用connexion.problem返回错误。

# 安全验证

#### OAuth 2认证和授权

Connexion支持三种OAuth 2处理方法中的一种。(see "TODO" below)使用Connexion，API安全定义必须包含`x-tokenInfoUrl`或`x-tokenInfoFunc（或分别设置TOKENINFO\_URL或TOKENINFO_FUNC env var respectively）`。`x-tokenInfoUrl`必须包含一个URL来验证和获取(token information)令牌信息，`x-tokenInfoFunc`必须包含对用于获取令牌信息的函数的引用。当使用`x-tokenInfoUrl`和`x-tokenInfoFunc`时，Connexion将优先考虑函数方法。 Connexion希望以RFC 6750第2.1节中描述的格式在授权标头字段中接收OAuth令牌。这一方面与通常的OAuth流程有显着的不同。

令牌信息响应的uid属性（用户名）将在用户参数中传递给处理函数。

您可以在Connexion的“examples”文件夹中找到minimal OAuth example application(最小的OAuth示例应用程序)。

#### HTTPS支持

当在API YAML文件中将HTTPS指定为方案时，所提供的Swagger UI中的所有URI都是HTTPS端点。问题：运行的默认服务器是“普通”HTTP服务器。这意味着Swagger UI不能用于使用API​​。使用Connexion时启动HTTPS服务器的正确方法是什么？

# Connexion食谱

本节旨在成为Connexion特定使用案例可能解决方案的食谱。

#### 自定义类型格式

可以根据请求参数和API的响应有效负载来定义要由Connexion有效负载验证使用的自定义类型格式。

假设您的API处理产品，并且您想要定义一个具有“金钱”格式值的字段price_label。您可以创建格式检查器函数并注册该函数以验证“金钱”格式的值。

可能在OpenAPI规范中定义的具有“货币”格式属性的产品可能模式示例：

```
type: object
properties:
  title:
    type: string
  price_label:
    type: string
    format: money
```

然后我们为这种类型的值创建一个格式检查器函数：

```
import re

MONEY_RE = re.compile('^\$\s*\d+(\.\d\d)?')

def is_money(val):
    if not isinstance(val, str):
        return True
    return MONEY_RE.match(val)
```

格式检查器函数预期在值与预期格式匹配时返回True，否则返回False。验证您尝试验证的值的类型是否与格式兼容也很重要。在我们的例子中，我们在执行任何进一步检查之前检查val是否是“string”类型。

使其工作的最后一步是将注册我们的is_money函数为json_schema库中的“money”格式。为此，我们可以使用draft4格式检查器装饰器

```
from jsonschema import draft4_format_checker

@draft4_format_checker.checks('money')
def is_money(val):
    ...
```

这就是您需要在Connexion应用程序中验证该格式的全部内容。请记住，在运行应用程序服务器之前，格式检查器应该被定义和注册。完整的示例可以在https://gist.github.com/rafaelcaricio/6e67286a522f747405a7299e6843cd93找到

#### CORS支持

CORS（跨域请求）不是内置到Connexion中的，但是您可以使用flask-cors库来设置CORS头文件：

```
import connexion
from flask_cors import CORS

app = connexion.FlaskApp(__name__)
app.add_api('swagger.yaml')

# add CORS support
CORS(app.app)

app.run(port=8080)
```

# 异常处理

#### 通过Flask处理程序呈现异常

Flask默认包含一个异常处理程序，该连接的应用程序可以使用add_error_handler方法进行代理。您可以挂钩状态代码或特定的异常类型。

Connexion正在从返回错误的Flask响应转到抛出异常，这是connexion.problem的一个子类。目前，在OAuth装饰器中抛出的异常已经被转换。

#### 默认异常处理

默认情况下，根据HTTP API的问题详细信息，连接异常是JSON序列化的

应用程序可以使用connexion.problem或从connexion.ProblemException和werkzeug.exceptions.HttpException子类继承的异常（例如werkzeug.exceptions.Forbidden）返回错误。这种情况的一个例子是connexion.exceptions.OAuthProblem异常

```
class OAuthProblem(ProblemException, Unauthorized):
    def __init__(self, title=None, **kwargs):
        super(OAuthProblem, self).__init__(title=title, **kwargs)
```

#### Examples of Custom Rendering Exceptions

要在启动connexion应用程序时自定义呈现异常，可以挂钩到自定义异常并以某种自定义格式呈现它。例如：

```
import connexion
from connexion.exceptions import OAuthResponseProblem

def render_unauthorized(exception):
    return Response(response=json.dumps({'error': 'There is an error in the oAuth token supplied'}), status=401, mimetype="application/json")

app = connexion.FlaskApp(__name__, specification_dir='./../swagger/', debug=False, swagger_ui=False)
app = app.add_error_handler(OAuthResponseProblem, render_unauthorized)

```

#### 自定义异常

连接中有几种异常类型，它们包含额外的信息以帮助您向用户呈现超出默认描述和状态代码的适当消息：

#### OAuthProblem
当Authorization Header存在某种验证问题时会抛出此异常

#### OAuthResponseProblem
当您的OAuth 2服务器出现验证问题时，会引发此异常。它包含一个token\_response属性，其中包含来自OAuth 2服务器的完整http响应

#### OAuthScopeProblem
此范围表示OAuth 2服务器未生成所有需要的范围的令牌。这包含3个属性

- required\_scopes - 此端点所需的作用域

- token\_scopes - 为此端点授予的作用域

- missing_scopes - 访问此端点所需的OAuth 2服务器未提供的作用域
