
.. highlight:: python
    :linenothreshold: 0

处理用户的模式
===========================

.. image:: _static/images/users.png
   :alt: Patterns for handling users

一个现代 web 应用程序需要做的最常见的事情就是处理用户。拥有基本账号功能的一个应用程序需要处理很多的事情，像注册，确认电子邮箱，安全地存储密码，安全地重置密码，认证等等。因为在处理用户的时候存在很多安全的问题，通常最佳的方式就是坚持在这个领域中的标准模式。

.. note::

   在本节中，我们假设你是使用 SQLAlchemy 模型以及 WTForms 来处理你的表单输入。如果你没有使用这些的话，你需要自己使你的首选方法适应这些模式。

确认电子邮箱
------------------

当一个新用户提供我们他们的邮箱，我们通常要确认他们提供给我们的邮箱是否是正确的。一旦我们已经通过邮箱验证，我们可以安心地给我们用户发送密码重置连接以及其它敏感的信息，而无需担心是谁在接收这些内容。

确认邮箱最常见的模式之一就是发送一个 URL 唯一的密码重置链接，当访问它的时候，证实了用户的邮箱地址。例如，john@gmail.com 注册了我们的应用程序。我们把他的用户数据插入到数据库中，该条用户数据的 ``email_confirmed`` 字段被设置成 ``False`` 并且发送了一封携带唯一的 URL 的邮件到 john@gmail.com 上。这个 URL 通常包含一个唯一的令牌，例如，*http://myapp.com/accounts/confirm-/Q2hhZCBDYXRsZXR0IHJvY2tzIG15IHNvY2tz*。当 John 收到这封邮件的时候，他点击链接。我们的应用程序会检查令牌，知道谁在确认邮箱并且设置 John 的 ``email_confirmed`` 字段为 ``True``。

我们是如何知道 URL 中的令牌是对应哪个用户？一种方式就是在令牌被创建的时候存储到数据库中，当我们收到确认请求的时候到数据库中检查对比。这是一个很大的开销，幸运地是，我们不必这么做。

我们会对邮箱地址编码成令牌。并且令牌也包含一个时间戳，该时间戳是让我们设置一个令牌在什么时间内有效的时间限制。为了完成这些，我们使用 ``itsdangerous`` 包。这个包提供我们一个用来发送敏感数据到一个不可信的环境的工具（像发送一封邮件确认令牌到一个未确认的邮箱）。在本例中，我们将会使用 ``URLSafeTimedSerializer`` 类的一个实例。

::

   # ourapp/util/security.py

   from itsdangerous import URLSafeTimedSerializer

   from .. import app

   ts = URLSafeTimedSerializer(app.config["SECRET_KEY"])

当一个用户给我们他们的邮箱地址的时候，我们可以使用它序列化来生成一个确认令牌。我们实现了一个简单的账号创建过程，里面就使用了这种方法。

::

   # ourapp/views.py

   from flask import redirect, render_template, url_for

   from . import app, db
   from .forms import EmailPasswordForm
   from .util import ts, send_email

   @app.route('/accounts/create', methods=["GET", "POST"])
   def create_account():
       form = EmailPasswordForm()
       if form.validate_on_submit():
           user = User(
               email = form.email.data,
               password = form.password.data
           )
           db.session.add(user)
           db.session.commit()

           # Now we'll send the email confirmation link
           subject = "Confirm your email"

           token = ts.dumps(self.email, salt='email-confirm-key')

           confirm_url = url_for(
               'confirm_email',
               token=token,
               _external=True)

           html = render_template(
               'email/activate.html',
               confirm_url=confirm_url)

           # We'll assume that send_email has been defined in myapp/util.py
           send_email(user.email, subject, html)

           return redirect(url_for("index"))

       return render_template("accounts/create.html", form=form)

我们上面定义的视图处理用户的创建以及发送一封邮件到指定的邮箱地址。你可能注意到我们使用了一个模板用来生成邮件内容的 HTML 形式。

::

   {# ourapp/templates/email/activate.html #}

   Your account was successfully created. Please click the link below<br>
   to confirm your email address and activate your account:

   <p>
   <a href="{{ confirm_url }}">{{ confirm_url }}</a>
   </p>

   <p>
   --<br>
   Questions? Comments? Email hello@myapp.com.
   </p>

好了，现在我们只需要实现一个处理邮件中确认链接的视图。

::

   # ourapp/views.py

   @app.route('/confirm/<token>')
   def confirm_email(token):
       try:
           email = ts.loads(token, salt="email-confirm-key", max_age=86400)
       except:
           abort(404)

       user = User.query.filter_by(email=email).first_or_404()

       user.email_confirmed = True

       db.session.add(user)
       db.session.commit()

       return redirect(url_for('signin'))

这个视图是一个简单的表单视图。我们只在开始的时候添加了 ``try ... except`` 来检查令牌是否有效。令牌中包含了一个时间戳，因此我们能够告诉 ``ts.loads()`` 引发一个异常如果它大于 ``max_age`` 的话。在本例中，我们设置 ``max_age`` 为 86400 秒，即：24小时。

.. note::

   你可以使用非常相似的方法来实现更新邮箱地址的功能。只要发送一封携带令牌的邮件到新的邮箱，该令牌包含旧的以及新的邮箱地址。如果令牌是有效的，用新的邮箱更新旧的邮箱。

存储密码
-----------------

处理用户的首要规则就是在存储密码之前用 Bcrypt（或者 scrypt，这里我们使用 Bcrypt）散列密码。我们绝不能明文存储密码。这是一个巨大的安全问题并且对于我们用户来说是不公平的。所有的这些辛勤工作都已经有人完成并且抽象出来给我们使用，所以没有理由不在这里遵循最佳实践。 

.. note::

   OWASP 是关于 Web 应用程序安全性的信息的业界最值得信赖的来源之一。看看一些他们 `关于安全编码的建议 <https://www.owasp.org/index.php/Secure_Coding_Cheat_Sheet#Password_Storage>`_。

我们将继续并且使用 Flask-Bcrypt 扩展在我们的应用中实现 bcrypt 包。这个包基本上是对 ``py-bcrypt`` 包的封装，但是为我们做了一些很烦人的事情（像在比较散列之前检查字符编码等等）。

::

    # ourapp/__init__.py

    from flask.ext.bcrypt import Bcrypt

    bcrypt = Bcrypt(app)

Bcrypt 算法强烈地被推荐的原因之一就是”未来的适应性“。这就意味着随着时间的推移，当计算能力变得越来越便宜的时候，我们可以把它变得越来越困难地被使用猜测上百万次密码这一种暴力方式来破解。我们使用越多的”循环“来散列密码，将会花费越多的时间来猜测。如果我们在存储密码之前使用算法散列密码 20 次的话，攻击者必须散列每一个它们的猜测 20 次。

请记住如果我们散列密码超过 20 次的话，我们的应用程序需要花费很长的一段时间来返回响应，具体要取决于什么时候处理完成。这就意味着当选择使用的”循环数“的时候，我们必须平衡安全和可用性。我们可以在给定时间内计算完成的”循环“取决于提供我们应用程序的计算资源。在 0.25 到 0.5 秒之间的时间内散列密码是一个很好的体验。我们应该尝试使用的”循环“至少为 12。

为了测试散列密码花费的时间，我们可以编写一个简单且快速的散列密码的 Python 脚本。

::

   # benchmark.py

   from flask.ext.bcrypt import generate_password_hash

   # Change the number of rounds (second argument) until it takes between
   # 0.25 and 0.5 seconds to run.
   generate_password_hash('password1', 12)

现在我们可以使用 UNIX 的 ``time`` 工具来记录时间的消耗数。

::

    $ time python test.py

    real    0m0.496s
    user    0m0.464s
    sys     0m0.024s

我做了一个快速的基准测试在一个小型的服务器上，12 ”循环“（rounds）是一个很合适的值，因此我们使用它来配置我的示例。

::

   # config.py

   BCRYPT_LOG_ROUNDS = 12

现在 Flask-Bcrypt 已经配置好了，是时候开始散列密码。我们可以在接收来自注册表单的请求的视图中手动去散列密码，但是我们必须在密码重置以及密码修改的视图中再次重复这样做。相反，我们要做的就是如何抽象散列，以便我们的应用程序无需我们考虑就能自己完成。这里我们会使用一个 **setter**，这样的话当我们设置 ``user.password = 'password1'`` 的话，在存储之前就会自动地使用 Bcrypt 散列密码。

::

   # ourapp/models.py

   from sqlalchemy.ext.hybrid import hybrid_property

   from . import bcrypt, db

   class User(db.Model):
       id = db.Column(db.Integer, primary_key=True, autoincrement=True)
       username = db.Column(db.String(64), unique=True)
       _password = db.Column(db.String(128))

       @hybrid_property
       def password(self):
           return self._password

       @password.setter
       def _set_password(self, plaintext):
           self._password = bcrypt.generate_password_hash(plaintext)

我们使用了 SQLAlchemy 的 hybrid 扩展来定义一个属性，这个属性从相同接口调用的时候拥有不同的功能。当我们为 ``user.password`` 属性赋值的时候，我们的 setter 就被调用。在它里面，我们散列一耳光明文的密码并且存储在用户表的 ``_password`` 字段中。因为我们使用 hybrid 属性，我们可以通过相同的 ``user.password`` 属性来访问散列的密码。

现在我们使用上面的模型为应用程序实现一个注册视图。

::

   # ourapp/views.py

   from . import app, db
   from .forms import EmailPasswordForm
   from .models import User

   @app.route('/signup', methods=["GET", "POST"])
   def signup():
       form = EmailPasswordForm()
       if form.validate_on_submit():
           user = User(username=form.username.data, password=form.password.data)
           db.session.add(user)
           db.session.commit()
           return redirect(url_for('index'))

       return render_template('signup.html', form=form)

认证
--------------

既然我们在数据库中有用户了，我们可以实现认证。我们要一个用户提交携带他们的用户名和密码的表单（尽管对一些应用来说这可能是邮箱和密码），接着确保他们是否提供了正确的密码。如果所有的都验证通过了，我们通过在他们的浏览器上设置一个 cookie 来标记他们已经通过认证。下一次他们再过来请求的时候我们通过查找 cookie 知道他们已经登录。 

让我们开始用 WTForms 定义一个 ``UsernamePassword`` 表单。

::

   # ourapp/forms.py

   from flask_wtf import Form
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired


   class UsernamePasswordForm(Form):
       username = StringField('Username', validators=[DataRequired()])
       password = PasswordField('Password', validators=[DataRequired()])

下一步我们在我们的用户模型中添加一个方法，该方法用来比较一个字符串和用户存储的散列密码。

::

   # ourapp/models.py

   from . import db

   class User(db.Model):

       # [...] columns and properties

       def is_correct_password(self, plaintext)
           return bcrypt.check_password_hash(self._password, plaintext)


Flask-Login
~~~~~~~~~~~

我们下一目标就是定义一个登录的视图，该视图用来服务和接收我们的表单。如果用户输入正确的凭证的话，我们将使用 Flask-Login 扩展来认证他们。这个扩展简化了处理用户会话和认证的过程。

我们需要的就是对 Flask-Login 进行一些小小的配置。

在 *\_\_init\_\_.py* 中，我们将定义 Flask-Login 的 ``login_manager``。

::

    # ourapp/__init__.py

    from flask.ext.login import LoginManager

    # Create and configure app
    # [...]

    from .models import User

    login_manager = LoginManager()
    login_manager.init_app(app)
    login_manager.login_view =  "signin"

    @login_manager.user_loader
    def load_user(userid):
        return User.query.filter(User.id==userid).first()

这里我们创建了一个 ``LoginManager`` 示例，并且用我们的 ``app`` 对象初始化它，定义登录视图并且告诉它如何用一个的用户的 ``id`` 得到用户对象。这是我们使用 Flask-Login 的最基本的配置。

.. note::

   查看更多 `自定义 Flask-Login 的方法 <https://flask-login.readthedocs.org/en/latest/#customizing-the-login-process>`_.

现在我们可以定义处理登录的 ``signin`` 视图。

::

   # ourapp/views.py

   from flask import redirect, url_for

   from flask.ext.login import login_user

   from . import app
   from .forms import UsernamePasswordForm()

   @app.route('signin', methods=["GET", "POST"])
   def signin():
       form = UsernamePasswordForm()

       if form.validate_on_submit():
           user = User.query.filter_by(username=form.username.data).first_or_404()
           if user.is_correct_password(form.password.data):
               login_user(user)

               return redirect(url_for('index'))
           else:
               return redirect(url_for('signin'))
       return render_template('signin.html', form=form)

我们简单地从 Flask-Login 中导入 ``login_user`` 函数，检查用户登录凭证并且调用 ``login_user(user)``。你可以使用 ``logout_user()`` 实现用户的退出操作。

::

   # ourapp/views.py

   from flask import redirect, url_for
   from flask.ext.login import logout_user

   from . import app

   @app.route('/signout')
   def signout():
       logout_user()

       return redirect(url_for('index'))

忘记密码
--------------------

我们通常要实现一个”忘记你的密码“的功能，允许一个用户通过邮箱找回自己的账号。这个地方也会有很多潜在的风险，因为关键是让一个未认证的用户接管一个账号。我们这里实现密码重置采用了我们在邮箱确认的时候一些同样的技术。

我们需要一个表单用来申请为某个账号的邮箱重置密码，并且需要一个表单就来让用书输入新的密码，一旦我们已经确认了未经认证的用户能够访问某个账号的邮箱。在本节的代码假设我们的用户模型有一个邮箱和密码，并且密码是我们之前创建的具有 hybrid 属性。

.. warning::

   不要发送密码重置链接到一个未经证实的电子邮件地址！你要确保你正在发送链接给合适的人。

我们将需要两个表单。一个是用于申请重置密码的链接，一个是用于一旦邮件被认证用于更改密码。

::

   # ourapp/forms.py

   from flask_wtf import Form
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired, Email

   class EmailForm(Form):
       email = TextField('Email', validators=[DataRequired(), Email()])

   class PasswordForm(Form):
       password = PasswordField('Email', validators=[DataRequired()])

上面的代码假设我们的密码重置的表单只需要一个密码字段（只需要输入一次新密码）。许多应用程序需要用户输入新的密码两次以确保他们没有输错。要做到这一点的话，我们可以简单地添加另一个 ``PasswordField`` 字段，并且添加 ``EqualTo`` WTForms 的验证器。 

.. note::

   用户体验社区（UX）有很多关于处理注册表单的最佳方式的有趣的讨论。我个人十分喜欢 Stack Exchange 用户（Roger Attrill）的想法，他这样说的：

   ”我们不应该要求用户输入密码两次 - 我们只需要用户输入一次并且确保‘忘记密码’的功能要完美和无缝的。“

   - 在 `Stack Exchange 用户体验跟帖 <http://ux.stackexchange.com/questions/20953/why-should-we-ask-the-password-twice-during-registration/21141>`_ 查看更多关于该话题的内容。

   - 在 `Smashing Magazine 的文章 <http://uxdesign.smashingmagazine.com/2011/05/05/innovative-techniques-to-simplify-signups-and-logins/>`_ 上也有很多关于简化注册和登录表单的很酷的想法。

现在我们实现第一个视图，用户可以申请发送密码重置链接到一个指定的邮箱地址。

::

   # ourapp/views.py

   from flask import redirect, url_for, render_template

   from . import app
   from .forms import EmailForm
   from .models import User
   from .util import send_email, ts

   @app.route('/reset', methods=["GET", "POST"])
   def reset():
       form = EmailForm()
       if form.validate_on_submit()
           user = User.query.filter_by(email=form.email.data).first_or_404()

           subject = "Password reset requested"

           # Here we use the URLSafeTimedSerializer we created in `util` at the
           # beginning of the chapter
           token = ts.dumps(user.email, salt='recover-key')

           recover_url = url_for(
               'reset_with_token',
               token=token,
               _external=True)

           html = render_template(
               'email/recover.html',
               recover_url=recover_url)

           # Let's assume that send_email was defined in myapp/util.py
           send_email(user.email, subject, html)

           return redirect(url_for('index'))
       return render_template('reset.html', form=form)

当表单接收到一个邮箱地址，我们获取与该邮箱地址有关的用户，生成一个重置的令牌并且发送他们一个密码重置的 URL。这个 URL 将他们路由到一个视图，该视图验证令牌并且让他们重置密码。

::

   # ourapp/views.py

   from flask import redirect, url_for, render_template

   from . import app, db
   from .forms import PasswordForm
   from .models import User
   from .util import ts

   @app.route('/reset/<token>', methods=["GET", "POST"])
   def reset_with_token(token):
       try:
           email = ts.loads(token, salt="recover-key", max_age=86400)
       except:
           abort(404)

       form = PasswordForm()

       if form.validate_on_submit():
           user = User.query.filter_by(email=email).first_or_404()

           user.password = form.password.data

           db.session.add(user)
           db.session.commit()

           return redirect(url_for('signin'))

       return render_template('reset_with_token.html', form=form, token=token)

我们使用了和验证用户的邮箱地址一样的令牌验证方法。视图把从 URL 中获取的令牌传入到模板中。接着模板使用令牌提交表单到正确的 URL。让我们看看模板可能的样子。

::

    {# ourapp/templates/reset_with_token.html #}

    {% extends "layout.html" %}

    {% block body %}
    <form action="{{ url_for('reset_with_token', token=token) }}" method="POST">
        {{ form.password.label }}: {{ form.password }}<br>
        {{ form.csrf_token }}
        <input type="submit" value="Change my password" />
    </form>
    {% endblock %}

摘要
-------

-  使用 itsdangerous 包来创建和验证发送到邮箱地址的令牌。
-  当一个用户创建账号，更改邮箱或者忘记密码的时候，你可以使用这些令牌来验证邮件。
-  使用 Flask-Login 扩展来认证用户可以避免自己处理一大堆麻烦的会话管理。
-  要经常思考一个恶意的用户如何滥用你的应用程序去做一些你不打算做的事情。
