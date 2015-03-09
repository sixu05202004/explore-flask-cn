
.. highlight:: python
    :linenothreshold: 0

处理表单
==============

.. image:: _static/images/forms.png
   :alt: Handling forms

表单是让用户与我们的网页应用程序交互的基本元素。Flask 本身并不会帮助我们处理表单，但是 Flask-WTF 扩展让我们在我们的 Flask 应用程序中使用流行的 WTForms 包。这个包使得定义表单和处理提交容易一些。

Flask-WTF
---------

我们想要使用 Flask-WTF 做的第一件事情（在安装它以后）就是在 ``myapp.forms`` 包中定义一个表单。

::

   # ourapp/forms.py

   from flask_wtf import Form
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired, Email

   class EmailPasswordForm(Form):
       email = StringField('Email', validators=[DataRequired(), Email()])
       password = PasswordField('Password', validators=[DataRequired()])

.. note::

   在 Flask-WTF 版本 0.9 以前，Flask-WTF 提供了针对 WTForms 字段以及验证器的自己的封装。你可能看到外面一大堆的代码是从 ``flask.ext.wtforms`` 中不是从 ``wtforms`` 中导入 ``TextField``，``PasswordField``。

   在 Flask-WTF 版本 0.9 以后，我们应该直接从 ``wtforms`` 中导入这些字段和验证器。

我们定义的表单是一个用户登录表单。我们把它叫做 ``EmailPasswordForm()``，我们可以重用这个同样的表单类（``Form``）去做其它的一些事情，像注册表单。这里我们没有去定义一个又长又没有用的表单，而是选择一个很常用的表单，只是为了给你们介绍使用 Flask-WTF 定义表单的方式。也许以后在正式项目中会定义一个特别复杂表单。对于表单中包含字段名称，我们建议使用一个清楚的名称，并且在一个表单中保持唯一。不得不说，对于一个长的表单，我们可能要给出一个更符合上文的字段名称。

登录表单可以替我们做一些事情。它能够保证我们应用程序的安全以防止 CSRF 漏洞，验证用户输入并且渲染适当的标记，这些标记是我们为表单定义的字段。

CSRF 保护和验证
~~~~~~~~~~~~~~~~~~

CSRF 表示跨站请求伪造。CSRF 攻击是指第三方伪造（像一个表单提交）请求到一个应用程序的服务器。一个易受攻击的服务器假设从一个表单来的数据是来自它自己的网站并且采取相应的操作。

作为一个例子，比方说，一个邮件提供商可以让你通过提交一个表单来删除你的账号。表单发送一个 POST 请求到服务器上的 ``account_delete`` 端点并且当表单被提交的时候删除登录的账号。我们可以在自己的网站上创建一个表单，该表单发送一个 POST 请求到同一个 `account_delete`` 端点。现在，如果我们让某人点击我们表单的提交按钮（或者通过 JavaScript 来这样做），邮件提供商提供的登录账号就会被删除掉。当然邮件提供商还不知道表单提交并不是发生在他们的网站上。

因此如何才能阻止 POST 请求来自别的网站？WTForms 通过在渲染每一个表单的时候生成一个唯一的令牌使得成为可能。生成的令牌会被传回到服务器，伴随着 POST 请求的数据，在表单被接受之前令牌必须接受服务器的验证。关键的是令牌是与存储在用户会话（cookies）的一个值有关并且会在一段时间后回去（默认是 30 分钟）。这种方式就能够保证提交一个有效表单的人就是加载页面的人（或者至少是使用同一电脑的人），而且他们只能在加载页面 30 分钟内这样做。

.. note::

   - `在文档中 <http://wtforms.simplecodes.com/docs/1.0.1/ext.html#module-wtforms.ext.csrf.session>`_ 了解更多关于 WTForms 如何生成这些令牌。

   - 在 `OWASP wiki <https://www.owasp.org/index.php/CSRF>`_ 中了解 CSRF。

要开始使用 Flask-WTF 保护 CSRF，我们需要为我们的登录页定义一个视图。

::

   # ourapp/views.py

   from flask import render_template, redirect, url_for

   from . import app
   from .forms import EmailPasswordForm

   @app.route('/login', methods=["GET", "POST"])
   def login():
       form = EmailPasswordForm()
       if form.validate_on_submit():

           # Check the password and log the user in
           # [...]

           return redirect(url_for('index'))
       return render_template('login.html', form=form)

我们从 ``forms`` 包中导入我们的表单并且在视图中实例化它。接着我们运行 ``form.validate_on_submit()``。如果表单被提交（例如，如果 HTTP 方法是 PUT 或者 POST）并且通过我们定义在 *forms.py* 中的验证器验证过，这个函数（``form.validate_on_submit()``）将会返回 ``True``。

.. note::

   - `关于 Form.validate_on_submit 的文档 <https://flask-wtf.readthedocs.org/en/latest/api.html#flask_wtf.Form.validate_on_submit>`_
   - `关于 Form.validate_on_submit 的源码 <https://github.com/lepture/flask-wtf/blob/v0.9.5/flask_wtf/form.py#L151>`_

如果表单已经被提交和验证的话，我们可以继续登录的逻辑。如果它没有被提交的话（例如，只是一个 GET 请求），我们就要把表单对象传递给我们的模板，以便它能够后被渲染。下面就是我们使用 CSRF 保护的时候模板的样子。

::

    {# ourapp/templates/login.html #}

    {% extends "layout.html" %}
    <html>
        <head>
            <title>Login Page</title>
        </head>
        <body>
            <form action="{{ url_for('login') }}" method="post">
                <input type="text" name="email" />
                <input type="password" name="password" />
                {{ form.csrf_token }}
            </form>
        </body>
    </html>

``{{ form.csrf_token }}`` 渲染了一个隐藏的字段，该字段包含那些奇特的 CSRF 令牌，并且当 WTForms 验证表单的时候会寻找这个字段。我们不用担心包含处理令牌的逻辑，WTForms 会主动帮我们去做。好哇！

使用 CSRF 令牌保护 AJAX 调用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flask-WTF CSRF 令牌不限于保护表单提交。如果你的应用程序要处理其它可能会被伪造的请求（特别是 AJAX 调用），你也可以在那里添加 CSRF 保护！

.. note::

    Flask-WTF 文档中谈到了更多地关于 `在 AJAX 调用中使用这些 CSRF 令牌 <https://flask-wtf.readthedocs.org/en/latest/csrf.html#ajax>`_。

自定义验证
~~~~~~~~~~~~~~~~~

除了由 WTForms 提供的内置的表单验证器（例如，``Required()``，``Email()`` 等等），我们能创建我们自己的验证器。我们将通过编写一个 ``Unique()`` 验证器来说明创建自己的验证器，``Unique()`` 验证器是用来检查数据库并且确保用户提供的值在数据库中不存在。这能够用于确保用户名或者邮箱地址还没有使用。没有 WTForms 的话，我们可能要在视图中做这些事情，但是现在我们可以在表单本身做些事情。

现在我们来定义一个简单的注册表单，其实这个表单和登录的表单几乎一样。只是会在后面给它添加一些自定义的验证器。

::

   # ourapp/forms.py

   from flask_wtf import Form
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired, Email

   class EmailPasswordForm(Form):
       email = StringField('Email', validators=[DataRequired(), Email()])
       password = PasswordField('Password', validators=[DataRequired()])

现在我们要添加我们的验证器用来确保它们提供的邮箱不存在数据库中。我们把这个验证器放在一个新的 ``util`` 模块，``util.validators``。

::

    # ourapp/util/validators.py
    from wtforms.validators import ValidationError

    class Unique(object):
        def __init__(self, model, field, message=u'This element already exists.'):
            self.model = model
            self.field = field

        def __call__(self, form, field):
            check = self.model.query.filter(self.field == field.data).first()
            if check:
                raise ValidationError(self.message)

这个验证器假设我们是使用 SQLAlchemy 来定义我们的模型。WTForms 期待验证器返回某种可调用的对象（例如，一个可调用的类）。

在 ``Unique()`` 的 ``__init__`` 中我们可以指定哪些参数传入到验证器中，在本例中我们要传入相关的模型（例如，在我们的情况中式传入 ``User`` 模型）以及要检查的字段。当验证器被调用的时候，如果定义模型的任何实例匹配表单中提交的值，它将会抛出一个 ``ValidationError``。我们也可以添加一个具有通用默认值的消息，它将会被包含在 ``ValidationError`` 中。

现在我们可以修改 ``EmailPasswordForm``，使用我们自定义的 ``Unique`` 验证器。

::

   # ourapp/forms.py

   from flask_wtf import Form
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired

   from .util.validators import Unique
   from .models import User

   class EmailPasswordForm(Form):
       email = StringField('Email', validators=[DataRequired(), Email(),
           Unique(
               User,
               User.email,
               message='There is already an account with that email.')])
       password = PasswordField('Password', validators=[DataRequired()])

.. note::

   我们的验证器不必须是一个可调用的类。它也可能是返回可调用或者直接调用的一个工厂模式。WTForms 文档中有 `一些例子 <http://wtforms.simplecodes.com/docs/0.6.2/validators.html#custom-validators>`_。

渲染表单
~~~~~~~~~~~~~~~
 
WTForms 也能帮助我们为表单渲染成 HTML 表示。WTForms 实现的 ``Field`` 字段能够渲染成该字段的 HTML 表示，所以为了渲染它们，我们只必须在我们模板中调用表单的字段。这就像渲染 ``csrf_token`` 字段。下面给出了一个登录模板的示例，在里面我们使用 WTForms 来渲染我们的字段。

::

    {# ourapp/templates/login.html #}

    {% extends "layout.html" %}
    <html>
        <head>
            <title>Login Page</title>
        </head>
        <body>
            <form action="" method="post">
                {{ form.email }}
                {{ form.password }}
                {{ form.csrf_token }}
            </form>
        </body>
    </html>

我们可以自定义如何渲染字段，通过传入字段的属性作为参数到调用中。

::

   <form action="" method="post">
       {{ form.email.label }}: {{ form.email(placeholder='yourname@email.com') }}
       <br>
       {{ form.password.label }}: {{ form.password }}
       <br>
       {{ form.csrf_token }}
   </form>

.. note::

   如果我们想要传入 “class” HTML 属性，我们必须使用 ``class_=''`` 因为 “class” 是 Python 中的保留关键字。

.. note::

   WTForms 文档中有一个 `可用字段属性列表 <http://wtforms.simplecodes.com/docs/1.0.4/fields.html#wtforms.fields.Field.name>`_。

.. note::

   你可能注意到我们没有必须要使用 Jinja 的 ``|safe`` 过滤器。这是因为 WTForms 渲染 HTML 安全字符串。

   更多的内容请参阅 `官方文档 <https://flask-wtf.readthedocs.org/en/v0.8.4/#using-the-safe-filter>`_。

摘要
-------

-  表单从安全性的角度是很可怕的。
-  WTForms（以及 Flask-WTF）使得容易地定义，保护以及渲染你的表单。
-  使用 Flask-WTF 提供的 CSRF 保护可以确保你的表单的安全。 
-  你也可以使用 Flask-WTF 来保护你的 AJAX 调用以防止 CSRF 攻击。
-  定义自定义的表单验证器可以让验证逻辑远离视图。
-  使用 WTForms 字段渲染来渲染你的表单的 HTML，在你对你的表单定义做出一些改变的时候，你不必每次都更新它。


