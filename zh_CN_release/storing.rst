
.. highlight:: python
    :linenothreshold: 0

存储数据
============

.. image:: _static/images/storing.png
   :alt: Storing data

大部分 Flask 应用程序会在某一时刻处理存储数据。存在许多不同的方式来存储数据。寻找最好的一种方式完全取决于你要存储的数据。如果你存储关系型数据（例如，一个用户有多篇文章，每篇文章都有一个作者等等），一个关系型数据库可能是一种合适的方式。其它类型的数据可能适合 NoSQL 数据存储，像 MongoDB。

我将不会告诉你们该如何为你的应用程序选择数据库引擎。有些人会告诉你 NoSQL 是唯一的选择，而有些人会告诉你的关系型数据是合适的选择。所有我想要说的就是如果你不确认如何选择的话，一个关系型数据库（MySQL，PostgreSQL等等）肯定能为你正在做的事情工作。

另外，当你使用一个关系型数据库的话，你就会开始使用 SQLAlchemy，SQLAlchemy 是很有趣的！

SQLAlchemy
----------

SQLAlchemy 是一个 ORM（对象关系映射）。它基本上是在我们的数据库中执行的原始的 SQL 查询的之上的抽象层。它为一个长长的列表的数据库引擎提供一致的 API。其中最流行的数据库引擎包括 MySQL, PostgreSQL 和 SQLite。它使得在我们模型和数据库之间很容易地移动数据，并且它真的很容易地去做一些其它的事情，比如更换数据库引擎和迁移我们的数据库模式。

有一个很伟大的 Flask 扩展使得可以容易地在 Flask 中使用 SQLAlchemy。它叫做 Flask-SQLAlchemy。Flask-SQLAlchemy 为 SQLAlchemy 配置大量完整的默认值。它也处理一些会话管理因此我们不需要在我们的应用程序代码中做一些清洁工作。

让我们深入到一些代码中。我们要定义一些模型接着做一些配置工作。模型会在 *myapp/models.py* 中编写，但是首先需要在 *myapp/__init__.py* 中定义我们的数据库。

::

    # ourapp/__init__.py

    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    app = Flask(__name__, instance_relative_config=True)

    app.config.from_object('config')
    app.config.from_pyfile('config.py')

    db = SQLAlchemy(app)

首先我们初始化以及配置我们的 Flask 应用程序，接着我们使用它来初始化我们的 SQLAlchemy 处理程序。我们将要使用一个实例文件夹来为我们的数据库配置，所以当我们初始化应用程序应该使用 ``instance_relative_config`` 选项，接着调用 ``app.config.from_pyfile`` 加载它。现在我们可以定义我们的模型。

::

   # ourapp/models.py

   from . import db 

   class Engine(db.Model):

       # Columns

       id = db.Column(db.Integer, primary_key=True, autoincrement=True)

       title = db.Column(db.String(128))

       thrust = db.Column(db.Integer, default=0)

``Column``，``Integer``，``String``，``Model`` 以及其它的 SQLAlchemy 类可以从由 Flask-SQLAlchemy 定义的 ``db`` 对象中引用。我们已经定义了一个模型用来存储我们飞行器引擎当前的状态。每一个飞行器引擎有一个 id，一个标题以及一个推力水平。

我们还需要把一些数据库信息添加到我们配置中。我们使用一个实例文件夹为了保持敏感的配置变量不在版本控制中，所以我们将要把数据库的配置放入 *instance/config.py*。

::

   # instance/config.py

   SQLALCHEMY_DATABASE_URI = "postgresql://user:password@localhost/spaceshipDB"

.. note::

   你的数据库 URI 会根据你使用的数据库引擎而不同。请参阅 `SQLAlchemy 官方文档 <http://docs.sqlalchemy.org/en/latest/core/engines.html?highlight=database#database-urls>`_ 获取更多的内容。

初始化数据库 
~~~~~~~~~~~~~~

现在数据库已经配置好了并且我们已经定义一个模型，我们可以初始化数据库。这一步基本上涉及到从模型定义中创建数据库模式。

通常这个过程可能十分痛苦。我们是幸运的，SQLAlchemy 有一个很酷的命令，它将替我们做了所有的事情。

让我们在我们的仓库/版本库的根目录下打开一个 Python 终端。

::

    $ pwd
    /Users/me/Code/myapp
    $ workon myapp
    (myapp)$ python
    Python 2.7.5 (default, Aug 25 2013, 00:04:04) 
    [GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.0.68)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> from myapp import db
    >>> db.create_all()
    >>>

现在多亏了 SQLAlchemy，我们的表已经在我们的配置中指定的数据库中创建好了。

Alembic 迁移数据库
~~~~~~~~~~~~~~~~~~~~~

数据库的模式不是一成不变的。例如，我们可能要给我们的飞行器引擎表添加一个 ``last_fired`` 字段。如果我们没有任何数据的话，我们只要更新模型和再次运行 ``db.create_all()``。然而，如果我们已经在表中记录了 6 个月的引擎的数据，我们可能不希望从头开始。这就是数据库迁移的工作。

Alembic 是一个专门为 SQLAlchemy 使用的数据库迁移工具。它能够让我们保持我们的数据库模式的版本历史，以便我们后面可以升级到一个新的模式以及甚至退回到一个旧的版本。

Alembic 有一个很广泛的教程，大家很容易入门，因此我只给你一个简单的概述并且指出几件需要注意的事项。

我们将使用 ``alembic init`` 命令行创建我们 alembic 的“迁移环境”。一旦我们在我们的仓库/版本库中运行这个命令，我们将会有一个新的文件夹，它有一个很有创意的名字：*alembic*。下面这个示例改编自 Alembic 教程。

::

    ourapp/
        alembic.ini
        alembic/
            env.py
            README
            script.py.mako
            versions/
                3512b954651e_add_account.py
                2b1ae634e5cd_add_order_id.py
                3adcc9a56557_rename_username_field.py
        myapp/
            __init__.py
            views.py
            models.py
            templates/
        run.py
        config.py
        requirements.txt


*alembic/* 目录下有在不同版本之间我们数据迁移的脚本。同样也有包含配置信息的一个 *alembic.ini* 文件。

.. note::

    请把 *alembic.ini* 添加到 *.gitignore* 中！你将会把你的数据库的凭证放在这个文件中，因此你 **不会** 要它出现在版本控制中。

    你想让 *alembic/* 在文本控制中。它并不包含敏感信息（它不可能从你的源代码中产生的）并且保持它在版本控制中意味着有多个副本。

当开始要做出一个模式改变，会有这些步骤要走：首先我们运行 ``alembic revision`` 来生成一个迁移脚本。接着我们在 *myapp/alembic/versions/* 中打开一个新生成的 Python 文件并且使用 Alembic 的 ``op`` 对象提供的工具填充 ``upgrade`` 和 ``downgrade`` 函数。

一旦我们已经准备好我们的迁移脚本，我们可以运行 ``alembic upgrade head`` 来迁移我们的数据到最新的版本。

.. note::

   关于配置 Alembic，创建你的迁移脚本以及运行你的迁移的细节，请参阅 `Alembic 教程 <http://alembic.readthedocs.org/en/latest/tutorial.html>`_。

.. warning::

   别忘记制定一个计划在合适的时间备份你的数据。计划的细节不在本书范围内，但是你应该一直以一种安全可靠的方式备份数据库。

.. note::

   NoSQL 场景很少在 Flask 中使用，但是只要你选择的数据库引擎有一个 Python 库，你应该能够使用它。甚至有不少 `Flask 扩展 <http://flask.pocoo.org/extensions/>`_ 帮助整合 Flask 和 NoSQL 引擎。

摘要
-------

-  使用 SQLAlchemy 与关系型数据库一起工作。
-  使用 Flask-SQLAlchemy 与 SQLAlchemy 一起工作。
-  Alembic 帮助你在数据库模式的变化之间迁移你的数据。
-  你可以在 Flask 中使用 NoSQL，但是方法和工具与其它的数据库引擎迥异。
-  备份你的数据！

