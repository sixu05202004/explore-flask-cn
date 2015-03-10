
.. highlight:: python
    :linenothreshold: 0

配置
=============

.. image:: _static/images/configuration.png
   :alt: Configuration

当你学习 Flask 的时候，配置看起来很简单。你只要在 *config.py* 中定义一些变量接着一切就能工作了。当你开始必须要管理生产应用的配置的时候，这些简单性开始消失了。你可能需要保护 API 密钥以及为不同的环境使用不同的配置（例如，开发和生产环境）。在本章节中我们会介绍 Flask 一些先进的功能，它可以使得管理配置容易些。

简单的例子
---------------

一个简单的应用程序可能不会需要任何这些复杂的功能。你可能只需要把 *config.py* 放在你的仓库/版本库的根目录并且在 *app.py* 或者 *yourapp/\_\_init\_\_.py* 中加载它。

*config.py* 文件中应该每行包含一个配置变量赋值。当你的应用程序初始化的时候，在 *config.py* 中的配置变量用于配置 Flask 和它的扩展并且它们能够通过 ``app.config`` 字典访问到 -- 例如，``app.config["DEBUG"]``。

::

   DEBUG = True # Turns on debugging features in Flask
   BCRYPT_LEVEL = 12 # Configuration for the Flask-Bcrypt extension
   MAIL_FROM_EMAIL = "robert@example.com" # For use in application emails

配置的变量可以被 Flask，它的扩展或者你来使用。这个例子中， 每当我们在一封事务性邮件中需要默认的 “发件人” 的时候，我们可以使用 ``app.config["MAIL_FROM_EMAIL"]`` -- 例如，密码重置。把这些信息放置于一个配置变量中使得以后能够容易地修改它。

::

    # app.py or app/__init__.py
    from flask import Flask

    app = Flask(__name__)
    app.config.from_object('config')

    # Now we can access the configuration variables via app.config["VAR_NAME"].


============== ====================================================  ==============================
变量           描述                                                  建议
============== ====================================================  ==============================
DEBUG          为你提供了调试错误的一些方便的工具。                  开发环境中设置成 ``True``；
               这包括一个基于 Web 的堆栈跟踪和交互式的               生产环境中设置成 ``False``。
               Python 控制台。

SECRET_KEY     这是 Flask 用来为 cookies 签名的密钥。                这应该是一个复杂的随机值。  
               它也能被像 Flask-Bcrypt 类的扩展使用。
               你应该在你的实例文件夹中定义它，
               这样可以远离版本控制。
               你可以在下一个章节中阅读更多关于示例文件夹的内容。

BCRYPT_LEVEL   如果你使用 Flask-Bcrypt 来散列用户密码的话，          后面我们会给出在 Flask 应用中
               你需要指定一个“循环”数，这个数是在执行散列密码的      使用 Bcrypt 的一些最佳实践。
               算法需要的。如果你不使用 Flask-Bcrypt，你可以
               忽略这里。用于散列密码的循环数越大，攻击者猜测密码
               的时间会越长。同时，循环数越大会增加散列密码的时间。
============== ====================================================  ==============================

.. warning::

   确保在生产环境中 ``DEBUG`` 设置成 ``False``。如果保留 ``DEBUG`` 为 ``True``，它允许用户在你的服务器上执行任意的 Python。

实例文件夹
---------------

有时候你需要定义包含敏感信息的配置变量。我们想要从 *config.py* 中分离这些变量并且让它们保留在仓库/版本库之外。你可能会隐藏像数据库密码以及 API 密钥的一些敏感信息，或者定义于特定于指定机器的配置变量。为让实现这些要求更加容易些，Flask 提供了一个叫做 **instance folders** 的功能。实例文件夹是仓库/版本库下的一个子目录并且包含专门为这个应用程序的实例的一个配置文件。我们不希望它提交到版本控制。

::

    config.py
    requirements.txt
    run.py
    instance/
      config.py
    yourapp/
      __init__.py
      models.py
      views.py
      templates/
      static/

使用实例文件夹
~~~~~~~~~~~~~~~~~~~~~~

我们使用 ``app.config.from_pyfile()`` 来从一个实例文件夹中加载配置变量。当我们调用 ``Flask()`` 来创建我们的应用的时候，如果我们设置了 ``instance_relative_config=True``，``app.config.from_pyfile()`` 将会从 *instance/* 目录加载指定文件。

::

    # app.py or app/__init__.py

    app = Flask(__name__, instance_relative_config=True)
    app.config.from_object('config')
    app.config.from_pyfile('config.py')

现在我们可以像在 *config.py* 中那样在 *instance/config.py* 中定义配置变量。你也应该把你的实例文件夹加入到版本控制系统的忽略列表中。要使用 Git 做到这一点的话，你需要在 *.gitignore* 新的一行中添加 ``instance/`` 。

密钥
~~~~~~~~~~~

实例文件夹的隐私性成为在其里面定义不想暴露到版本控制的密钥的最佳候选。这些密钥可能包含了你的应用的密钥或者第三方 API 密钥。如果你的应用是开源的或者以后可能会公开的话，这一点特别重要。我们通常要求其他用户或者贡献者使用自己的密钥。

::

   # instance/config.py

   SECRET_KEY = 'Sm9obiBTY2hyb20ga2lja3MgYXNz'
   STRIPE_API_KEY = 'SmFjb2IgS2FwbGFuLU1vc3MgaXMgYSBoZXJv'
   SQLALCHEMY_DATABASE_URI= \
   "postgresql://user:TWljaGHFgiBCYXJ0b3N6a2lld2ljeiEh@localhost/databasename"

基于环境的配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果在你的生产环境和开发环境中的差异非常小的话，你可能想要使用实例文件夹来处理配置的变化。定义在 *instance/config.py* 文件中的配置变量能够覆盖 *config.py* 中的值。你只需要在 ``app.config.from_object()`` 后调用 ``app.config.from_pyfile()``。这样用法的好处之一就是在不同的机器上修改你的应用的配置。

::

   # config.py

   DEBUG = False
   SQLALCHEMY_ECHO = False


   # instance/config.py
   DEBUG = True
   SQLALCHEMY_ECHO = True

在生产环境上，我们略去上面 *instance/-config.py* 中的配置变量，它会退回到定义在 *config.py* 中的值。

.. note::

   - 了解更多关于 Flask-SQLAlchemy 的 `配置项 <http://pythonhosted.org/Flask-SQLAlchemy/config.html#configuration-keys>`_。（中文版的位于：http://www.pythondoc.com/flask-sqlalchemy/config.html#configuration-keys）

基于环境变量配置
------------------------------------------

实例文件夹不应该出现在版本控制中。这就意味着你将无法跟踪你的实例配置的变化。如果只是一、两个变量这可能不是什么问题，但是如果你在不同的环境上（生产，预升级，开发，等等）配置都有些微调话，你就不会想要存在丢失它们的风险。

Flask 给我们选择配置文件的能力，它可以基于一个环境变量的值来加载不同的配置文件。这就意味着在我们的仓库/版本库里，我们可以有多个配置文件并且总会加载正确的那一个。一旦我们有多个配置文件的话，我可以把它们移入它们自己 ``config`` 文件夹中。

::

    requirements.txt
    run.py
    config/
      __init__.py # Empty, just here to tell Python that it's a package.
      default.py
      production.py
      development.py
      staging.py
    instance/
      config.py
    yourapp/
      __init__.py
      models.py
      views.py
      static/
      templates/

在上面的文件列表中我们有多个不同的配置文件。

======================= ==========================================================================
config/default.py       默认的配置值，可用于所有的环境或者被个人的环境给覆盖。

config/development.py   用于开发环境的配置值。这里你可能会指定本地数据库的 URI。

config/production.py    用于生产环境的配置值。在这里 ``DEBUG`` 一定要设置成 ``False``。

config/staging.py       根据开发进度，你可能会有一个模拟生产环境，这个文件主要用于这种场景。      
======================= ==========================================================================

为了决定要加载哪个配置文件，我们会调用 ``app.config.from_envvar()``。

::

    # yourapp/__init__.py

    app = Flask(__name__, instance_relative_config=True)

    # Load the default configuration
    app.config.from_object('config.default')

    # Load the configuration from the instance folder
    app.config.from_pyfile('config.py')

    # Load the file specified by the APP_CONFIG_FILE environment variable
    # Variables defined here will override those in the default configuration
    app.config.from_envvar('APP_CONFIG_FILE')

环境变量的值应该是配置文件的绝对路径。

我们如何设置这个环境变量取决于我们运行应用所在的平台。如果我们运行在一个普通的 Linux 服务器上，我们可以编写一个设置环境变量的 shell 脚本并且运行 *run.py*。

::

   # start.sh

   APP_CONFIG_FILE=/var/www/yourapp/config/production.py
   python run.py

*start.sh* 对于每一个环境都是独一无二的，因此它应该被排除在版本控制之外。在 Heroku 上，我们需要使用 Heroku 工具来设置环境变量。这种设置方式也适用于其它的 PaaS 平台。

摘要
-------

-  一个简单的应用程序可能仅仅需要一个配置文件：*config.py*。
-  实例文件夹能够帮助我们隐藏敏感的配置值。
-  实例文件夹能够用于针对每个特定的环境需要改变应用程序的配置的场景。
-  我们应该在一些复杂的以及多个的环境中使用环境变量以及 ``app.config.from_envvar()`` 。
