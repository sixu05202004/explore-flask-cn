
.. highlight:: python
    :linenothreshold: 0

蓝图
==========

.. image:: _static/images/blueprints.png
   :alt: Blueprints

什么是蓝图？
--------------------

一个蓝图定义了视图，模板，静态文件以及可以用于应用程序的其它元素的集合。例如，让我们假设下我们有一个管理面板的蓝图。这个蓝图会定义一些包含像 */admin/login* 和 */admin/dashboard* 路由的视图。它也可能包含服务于这些路由的模板以及静态文件。接着我们可以使用这个蓝图添加一个管理面板到我们的应用程序中，不论我们的应用程序是什么类型的。

为什么要使用蓝图？
-----------------------------

蓝图“杀手级”使用场景就是把我们的应用程序组织成不同的组件。对于一个类似 Twitter 的微型博客，我们可能有一个针对网站页面的蓝图，例如，*index.html* 和 *about.html*。接着我们还有另外一个带有登录面板的蓝图，在那里我们显示了所有最新的文章，然后我们还有一个用于后台管理的面板的蓝图。网站的每一个不同的区域也能够被分成不同区域的代码来实现。这能够让我们用几个小的 “apps” 构建我们的应用程序，每一个 apps 都在做一件事情。

.. note::

    请从 Flask 官方文档中 `"为什么使用蓝图" <http://flask.pocoo.org/docs/blueprints/#why-blueprints>`_ 阅读更多地使用蓝图的好处（中文版位于：http://www.pythondoc.com/flask/blueprints.html#id2）。

你把它们放哪里？
----------------------

使用蓝图组织我们的应用程序有很多的方式。通常情况下，我们可以考虑按功能结构和分区这两种选择（功能结构和分区这两个词语我借鉴了商业上的概念）。

功能结构
~~~~~~~~~~~~~~~~~~~~

按照功能结构的话，你可以通过它们所做的事情来组织你的应用程序的结构。模板在一个文件夹中，静态文件在另一个文件夹中，视图在第三个文件夹中。

::

    yourapp/
        __init__.py
        static/
        templates/
            home/
            control_panel/
            admin/
        views/
            __init__.py
            home.py
            control_panel.py
            admin.py
        models.py

除了 *yourapp/views/\_\_init\_\_.py*，在上面列表中的 *yourapp/views/* 文件夹中的每一个 *.py* 文件都是一个蓝图。在 *yourapp/\_\_init\_\_.py* 中我们要导入这些蓝图并且在我们的 ``Flask()`` 对象中 **注册** 它们。我们会在本章的后面看看实现方式。

.. note::

    Flask 站点：`http://flask.pocoo.org <http://flask.pocoo.org>`_ 使用的就是这种结构。你自己可以到 `GitHub <https://github.com/mitsuhiko/flask-website/tree/master/flask_website>`_ 上一睹真容。

分区
~~~~~~~~~~

对于分区结构了，你可以基于它们有助于应用程序的哪一部分来组织应用程序的结构。管理面板所有的模板，视图以及静态文件都在一个文件夹中，用户控制的所有的模板，视图和静态文件在另一个文件夹中。

::

    yourapp/
        __init__.py
        admin/
            __init__.py
            views.py
            static/
            templates/
        home/
            __init__.py
            views.py
            static/
            templates/
        control_panel/
            __init__.py
            views.py
            static/
            templates/
        models.py

像上面列出的应用程序的分区结构，在 *yourapp/* 中每一个文件夹都是一个单独的蓝图。所有的这些蓝图都会应用到顶层 *\_\_init\_\_.py* 中的 ``Flask()`` 对象中。

哪一个是最好的？
~~~~~~~~~~~~~~~~~~

你选择的组织结构很大程度上是一种个人决定。唯一的区别是层次结构的表示方式，因此你可以自由地决策要使用的组织结构，你可以选择一个对自己有意义的。

如果你的应用程序大部分是独立的结构，仅仅共享着像模型和配置，分区结构就是合适的选择方式。一个例子就是让用户建立网站的 SaaS 应用程序。你可能就有蓝图分别针对主页，控制面板，用户网站，以及管理面板。这些组件可能有完全不同的静态文件和布局。如果考虑负责这个应用程序或者分拆/重构这个应用程序的话，分区结构会更加适用一些。

另一方面，如果你的应用程序联系地更加紧密一些的话，它可能用一个功能结构呈现更加合适。一个示例就是 Facebook。如果 Facebook 使用 Flask 的话，它可能就有静态页（例如，登录-注销页，注册，关于等等），控制面板（例如，新闻源），个人主页（*/robert/about* 以及
*/robert/photos*），设置（*/settings/security* 和
*/settings/privacy*）等等一些蓝图。这些组件共享一个通用的布局和样式，但是每一个也会有自己的布局。下面的列表中展示了一个进行大量删减版的 Facebook 的样子，如果它是使用 Flask 构建的话。

::

    facebook/
        __init__.py
        templates/
            layout.html
            home/
                layout.html
                index.html
                about.html
                signup.html
                login.html
            dashboard/
                layout.html
                news_feed.html
                welcome.html
                find_friends.html
            profile/
                layout.html
                timeline.html
                about.html
                photos.html
                friends.html
                edit.html
            settings/
                layout.html
                privacy.html
                security.html
                general.html
        views/
            __init__.py
            home.py
            dashboard.py
            profile.py
            settings.py
        static/
            style.css
            logo.png
        models.py

在 *facebook/views/* 中的蓝图仅仅是视图的集合而不是完全独立的组件。同一的静态文件将会被大多数的蓝图的视图使用。大多数模板都会扩展一个主模板。功能结构是组织这个项目的一种好的方式。

你如何使用它们？
--------------------

基本用法
~~~~~~~~~~~

让我们看看 Facebook 示例中的其中一个蓝图的代码。

::

    # facebook/views/profile.py

    from flask import Blueprint, render_template

    profile = Blueprint('profile', __name__)

    @profile.route('/<user_url_slug>')
    def timeline(user_url_slug):
        # Do some stuff
        return render_template('profile/timeline.html')

    @profile.route('/<user_url_slug>/photos')
    def photos(user_url_slug):
        # Do some stuff
        return render_template('profile/photos.html')

    @profile.route('/<user_url_slug>/about')
    def about(user_url_slug):
        # Do some stuff
        return render_template('profile/about.html')

要创建一个蓝图对象，我们先导入 ``Blueprint()`` 类并且用参数 ``name`` 和 ``import_name`` 初始化它。通常情况下，``import_name`` 就是 ``__name__``，这是一个包含当前模块名称的特殊 Python 变量。

在这个 Facebook 示例中我们使用了一个功能结构。如果我们使用分区结构的话，我们要通知 Flask 蓝图有自己的模板和静态文件夹。此块的代码大概的样子如下所示。

::

    profile = Blueprint('profile', __name__,
                        template_folder='templates',
                        static_folder='static')

现在我们已经定义我们的蓝图。是时候在我们的 Flask 应用程序中注册它。

::

    # facebook/__init__.py

    from flask import Flask
    from .views.profile import profile

    app = Flask(__name__)
    app.register_blueprint(profile)

现在定义在 *facebook/views/profile.py* 上的路由（例如，``/<user_url_slug>``）在应用程序上注册并且表现得像你使用 ``@app.route()`` 定义它们一样。

使用动态的 URL 前缀
~~~~~~~~~~~~~~~~~~~~~~~~~~

继续 Facebook 例子，注意到所有的用户资料路由都是以 ``<user_url_slug>`` 开始并且把它的值传递给视图。我们希望用户们能够通过浏览像 *https://facebo-ok.com/john.doe* 类似的网址访问用户资料页。我们可以通过为所有的蓝图的路由定义一个动态的前缀来停止重复工作。

蓝图可以让我们定义动态和静态的前缀。我们可以通知 Flask 在一个蓝图中的所有的路由都是以 */profile* 为前缀的（这里的 */profile* 只是一个示例），这就是一个静态的前缀。至于 Facebook 示例，前缀是基于浏览的用户资料而变化。无论他们浏览哪个用户的个人资料，我们都应该在 URL 标签中显示。这就是一个动态的前缀。

我们可以选择在什么时候定义我们的前缀。我们可以在两个地方中的任意一个定义前缀：当我们实例化 ``Blueprint()`` 类或者当我们用 ``app.register_blueprint()`` 注册它的时候。

::

    # facebook/views/profile.py

    from flask import Blueprint, render_template

    profile = Blueprint('profile', __name__, url_prefix='/<user_url_slug>')

    # [...]

::

    # facebook/__init__.py

    from flask import Flask
    from .views.profile import profile

    app = Flask(__name__)
    app.register_blueprint(profile, url_prefix='/<user_url_slug>')

尽管没有任何技术因素限制任何一种方法，最好是在注册的时候统一定义可用的前缀。这使得以后修改或者调整更加容易和方便些。因为这个原因，我建议在注册的时候设置 ``url_prefix``。

我们可以在动态前缀中使用转换器，就像在 ``route()`` 调用中一样。这个也包含了我们自定义的转换器。当使用了转换器，我们可以在把前缀交给视图之前进行预处理。在这个例子中我们要基于传入到我们用户资料蓝图的 URL 中的 user_url_slug 来获取用户对象。这里我们需要使用 ``url_value_preprocessor()`` 装饰一个函数来完成这个需求。

::

    # facebook/views/profile.py

    from flask import Blueprint, render_template, g

    from ..models import User

    # The prefix is defined on registration in facebook/__init__.py.
    profile = Blueprint('profile', __name__)

    @profile.url_value_preprocessor
    def get_profile_owner(endpoint, values):
        query = User.query.filter_by(url_slug=values.pop('user_url_slug'))
        g.profile_owner = query.first_or_404()

    @profile.route('/')
    def timeline():
        return render_template('profile/timeline.html')

    @profile.route('/photos')
    def photos():
        return render_template('profile/photos.html')

    @profile.route('/about')
    def about():
        return render_template('profile/about.html')

我们使用 ``g`` 对象来存储用户对象并且 ``g`` 可以在 Jinja2 模板中使用。这就意味着对于实现一个极其简单的系统的话，我们现在要做的就是在视图中渲染模板。

::

    {# facebook/templates/profile/photos.html #}

    {% extends "profile/layout.html" %}

    {% for photo in g.profile_owner.photos.all() %}
        <img src="{{ photo.source_url }}" alt="{{ photo.alt_text }}" />
    {% endfor %}

.. note::

   - Flask 官方文档中有一篇关于使用前缀为国际化你的 URLs 的 `一个伟大的教程 <http://flask.pocoo.org/docs/patterns/urlprocessors/#internationalized-blueprint-urls>`_ 。

使用动态的子域（subdomain）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

许多 SaaS（软件即服务）的应用程序目前提供用户一个子域，用户可以使用这个子域来访问他们的软件。例如，Harvest 是一个时间追踪管理应用程序，它允许你从 yourname.harvestapp.com 访问你的控制面板。这里我将向你展示如何使用 Flask 处理像 Harvest 一样自动生成的子域。

对于这一部分，我们将要使用允许用户创建他们自己的网站的应用程序示例。假设我们的应用程序有三个蓝图，它们分别用于用户登录的主页，用户构建他们的网站的用户管理面板以及用户的网站。由于这三部分是不相关的，我们用分区结构来组织结构。

::

    sitemaker/
        __init__.py
        home/
            __init__.py
            views.py
            templates/
                home/
            static/
                home/
        dash/
            __init__.py
            views.py
            templates/
                dash/
            static/
                dash/
        site/
            __init__.py
            views.py
            templates/
                site/
            static/
                site/
        models.py

下面的表格展示了本应用程序中所有的蓝图。

============================== =============== ====================================================
URL                            Route           Description 
============================== =============== ====================================================
sitemaker.com                  sitemaker/home  只是一个普通的蓝图。围绕 *index.html*，*about.html* 
                                               以及 *pricing.html* 的视图，模板以及静态文件。

bigdaddy.sitemaker.com         sitemaker/site  这个蓝图使用一个动态的子域并且包含用户网站的元素。
                                               我们会在下面介绍一些用于实现这个蓝图的代码。

bigdaddy.sitemaker.com/admin   sitemaker/dash  这个蓝图使用了一个动态的子域和一个 URL 前缀。
============================== =============== ====================================================

我们可以用定义我们 URL 前缀同样的方式来定义我们的动态子域。两个选择：在蓝图文件夹或者在顶层的 *\_\_init\_\_.py* 中都是可用的，但是我们坚持再一次把它定义在 *sitemaker/\_\_init.py\_\_* 中。

::

    # sitemaker/__init__.py

    from flask import Flask
    from .site import site

    app = Flask(__name__)
    app.register_blueprint(site, subdomain='<site_subdomain>')

因为我们使用了分层结构，我们会在 *sitema-ker/site/\_\_init\_\_.py* 中定义蓝图。

::

    # sitemaker/site/__init__py

    from flask import Blueprint

    from ..models import Site

    # Note that the capitalized Site and the lowercase site
    # are two completely separate variables. Site is a model
    # and site is a blueprint.

    site = Blueprint('site', __name__)

    @site.url_value_preprocessor
    def get_site(endpoint, values):
        query = Site.query.filter_by(subdomain=values.pop('site_subdomain'))
        g.site = query.first_or_404()

    # Import the views after site has been defined. The views
    # module will needto import 'site' so we need to make
    # sure that we import views after site has been defined.
    import .views

现在我们从数据库中获取了站点信息，我们将会把用户的站点展示给正在请求他们子域的访问者。

为了让 Flask 能和子域一起工作，我们将需要指定 ``SERVER_NAME`` 配置变量。

::

   # config.py

   SERVER_NAME = 'sitemaker.com'

.. note::

   几分钟以前，当我正在起草这一章节的时候，有人在 IRC 说他们的子域在开发环境上工作正常，但是在生产环境上不正常。我问他们是否已经配置 ``SERVER_NAME``，事实证明他们在开发环境上设置但是没有在生产环境上配置。在生产环境上设置了 ``SERVER_NAME`` 解决他们的问题。

   在 `http://dev.pocoo.org/irclogs/%23pocoo.2013-07-30.log <http://dev.pocoo.org/irclogs/%23pocoo.2013-07-30.log>`_ 上可以看到我和 aplavin 之间的对话。

   我觉得这是足够巧合的，并且值得列入本节。

.. note::

    你可以同时设置一个子域和前缀。这里大家可以考虑考虑如何配置它们。

使用蓝图重构小的应用程序
----------------------------------------

我们将会介绍把一个应用程序重构成使用蓝图的步骤。我们选择一个很典型的 Flask 应用程序并且重构它。

::

    config.txt
    requirements.txt
    run.py
    U2FtIEJsYWNr/
      __init__.py
      views.py
      models.py
      templates/
      static/
    tests/

*views.py* 文件已经增长到 10,000 行的代码！我们一直在拖延重构它的时间，但是现在是时候重构。*views.py* 文件包含我们网站每一部分的视图。这些部分分别是主页，用户控制面板，管理控制面板，API 和公司的博客。

步骤 1：分区或者功能？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个应用是有完全不同的部分组成。例如，用户控制面板和公司博客之间的模板和静态文件是完全不共享的。我们将选择分区结构。

步骤 2：移动一些文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::

   在你对你的应用程序做出任何改变之前，都应该提交到版本控制中。你也不想不小心删除一些东西吧。

下一步我们将继续前进，并且为我们新的应用程序创建目录树。我们可以在一个包目录里为每一个蓝图创建一个文件夹。接着我们将完整地复制 *views.py*，*static/* 和 *templates/* 到每个蓝图目录。最后，我们可以从顶层包目录中删除它们（*views.py*，*static/* 和 *templates/*）。

::

    config.txt
    requirements.txt
    run.py
    U2FtIEJsYWNr/
      __init__.py
      home/
        views.py
        static/
        templates/
      dash/
        views.py
        static/
        templates/
      admin/
        views.py
        static/
        templates/
      api/
        views.py
        static/
        templates/
      blog/
        views.py
        static/
        templates/
      models.py
    tests/

步骤 3：废话少说
~~~~~~~~~~~~~~~~~~~~

现在我们可以到每一个蓝图目录中去删除那些不属于该蓝图的视图，静态文件和模板。你如何做这一步很大程度上取决你的应用程序是如何组织结构的。

最终的结果就是每一个蓝图只有一个 *views.py* 文件，并且 *views.py* 文件内的函数只适用于本蓝图。没有两个蓝图会为同一个路由定义一个视图。每一个 *templates/* 目录只包含在本蓝图的视图中使用的模板。每一个 *static/* 目录应该只包含有本蓝图使用的静态文件。

.. note::

   特别地注意：需要减少所有不必要的导入。这是很容易忘记的事情，最乐观的情况下它只会让你的代码显得有些混论，但是最差情况下，它们拖慢你的应用程序。

步骤 4：蓝图
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这是我们把我们的目录转变成为蓝图的关键一步。关键就是在 *\_\_init\_\_.py* 文件。首先，我们看看 API 蓝图的定义。

::

    # U2FtIEJsYWNr/api/__init__.py

    from flask import Blueprint

    api = Blueprint(
        'site',
        __name__,
        template_folder='templates',
        static_folder='static'
    )

    import .views

接下来我们在 U2FtIEJsYWNr 包顶层 *\_\_init\_\_.py* 文件里注册这个蓝图。

::

    # U2FtIEJsYWNr/__init__.py

    from flask import Flask
    from .api import api

    app = Flask(__name__)

    # Puts the API blueprint on api.U2FtIEJsYWNr.com.
    app.register_blueprint(api, subdomain='api')

确保路由是注册到蓝图上而不是应用程序（app）对象上。

::

   # U2FtIEJsYWNr/views.py

   from . import app

   @app.route('/search', subdomain='api')
   def api_search():
       pass

::

   # U2FtIEJsYWNr/api/views.py

   from . import api

   @api.route('/search')
   def search():
       pass

步骤 5：享受
~~~~~~~~~~~~~

现在我们应用程序比起它原来一个庞大的 *views.py* 文件已经是大大地模块化了。路由的定义十分简单，因为我们可以在每一个蓝图里面单独定义并且可以为每个蓝图像子域和 URL 前缀一样配置。

摘要
-------

-  一个蓝图定义了视图，模板，静态文件以及可以用于应用程序的其它元素的集合。
-  蓝图是组织你的应用程序的一种很好的方式。
-  在分区结构中，每一个蓝图是一个视图，模板，静态文件的集合，它们构成了应用程序的一部分。
-  在功能结构中，每一个蓝图只是视图的集合。所有的模板放在一起，静态文件也一样。
-  要使用一个蓝图，你首先需要定义它，接着通过调用 ``Flask.register_blueprint()`` 来注册它。
-  你可以定义一个动态的 URL 前缀，它能够用于在一个蓝图里所有的路由。
-  你也可以定义一个动态的子域，它能够用于一个蓝图里所有的路由。
-  使用蓝图重构一个越来越大的应用程序能够用 5 个小步骤来完成。

