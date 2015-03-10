
.. highlight:: python
    :linenothreshold: 0

静态文件
============

.. image:: _static/images/static.png
   :alt: Static files

顾名思义，静态文件就是那些不会改变的文件。在一般的应用程序中，静态文件包括 CSS 文件，JavaScript 文件以及图片。它们也可能是音频文件以及其它类似的东西。

组织你的静态文件
----------------------------

我们将会在我们的应用程序包里为我们的静态文件创建一个叫做 *static* 的目录。

::

    myapp/
        __init__.py
        static/
        templates/
        views/
        models.py
    run.py

在 *static/* 中如何组织你的文件是取决于个人喜好。就我个人而言，我会对第三方的库（比如，jQuery， Bootstrap等等）混在我们自己的 JavaScript 和 CSS 文件里感到困扰。为了避免这个，我建议把第三方库分离到相应目录里一个 *lib/* 文件夹。一些项目使用 *vendor/* 来代替 *lib/*。

::

   static/
       css/
           lib/
               bootstrap.css
           style.css
           home.css
           admin.css
       js/
           lib/
               jquery.js
           home.js
           admin.js
       img/
           logo.svg
           favicon.ico

添加一个 favicon
~~~~~~~~~~~~~~~~~

在我们静态目录的文件是可以通过 *example.com/static/* 访问的。默认情况下，网页浏览器以及其它软件期望我们的 favicon 是在 *example.com/favicon.ico*。为了解决这个矛盾，我们在网站模板中的 ``<head>`` 部分添加如下的内容。

::

   <link rel="shortcut icon"
       href="{{ url_for('static', filename='img/favicon.ico') }}">

使用 Flask-Assets 管理静态资源
--------------------------------------

Flask-Assets 是一个用来管理你的静态文件的扩展。Flask-Assets 提供了两个非常有用的工具。首先，它可以让你们在 Python 代码中定义资源的 **束/包（bundles）**，这些束/包（bundles）能够被一起插入到你的模板。其次，它可以让你们 **预处理（pre-process）** 这些文件。这就意味着你们能够合并和压缩你们的 CSS 和 JavaScript 文件，使得用户仅仅只需要加载两个压缩的文件（CSS 和 JavaScript），而不需要迫使你开发一个复杂的资源管道。你甚至可以编译来自 Sass, LESS, CoffeeScript 以及一堆其它来源的文件。

::

   static/
       css/
           lib/
               reset.css
           common.css
           home.css
           admin.css
       js/
           lib/
               jquery-1.10.2.js
               Chart.js
           home.js
           admin.js
       img/
           logo.svg
           favicon.ico

定义束/包（bundles）
~~~~~~~~~~~~~~~~~~~~~

我们的应用程序有两个部分：公共站点和管理面板，在我们的应用程序中分别称为 "home" 和 "admin"。我们将定义四个束/包（bundles）：为每一部分定义一个 JavaScript 和 CSS 束/包。我们将把它们放在我们的 ``util`` 包里的一个 assets 模块里。

::

   # myapp/util/assets.py

   from flask.ext.assets import Bundle, Environment
   from .. import app

   bundles = {

       'home_js': Bundle(
           'js/lib/jquery-1.10.2.js',
           'js/home.js',
           output='gen/home.js),

       'home_css': Bundle(
           'css/lib/reset.css',
           'css/common.css',
           'css/home.css',
           output='gen/home.css),

       'admin_js': Bundle(
           'js/lib/jquery-1.10.2.js',
           'js/lib/Chart.js',
           'js/admin.js',
           output='gen/admin.js),

       'admin_css': Bundle(
           'css/lib/reset.css',
           'css/common.css',
           'css/admin.css',
           output='gen/admin.css)
   }

   assets = Environment(app)

   assets.register(bundles)

Flask-Assets 按照它们被列出的次序来合并你的文件。如果 *admin.js* 需要 *jquery-1.10.2.js* 的话，确保 jquery 列在最前面。

为了更容易地注册，我们定义束/包在一个目录里。Flask-Assets 内部使用的 webassets 包让我们可以用多种方式来注册束/包（bundles），包括像我们在上面代码段中传入一个字典。[1]_

因为我们在 ``util.assets`` 中注册了我们的束/包（bundles），所有我们必须要做的就是在我们的应用程序初始化后在 *\_\_init\_\_.py* 中导入 ``assets``。

:: 

    # myapp/__init__.py

    # [...] Initialize the app

    from .util import assets

使用我们的束/包（bundles）
~~~~~~~~~~~~~~~~~~~~~~~~~~~

要使用我们的 “admin” 束/包（bundles），我们需要在 “admin” 部分的父模板：*admin/layout.html* 中插入它们。

::

   templates/
       home/
           layout.html
           index.html
           about.html
       admin/
           layout.html
           dash.html
           stats.html

::

    {# myapp/templates/admin/layout.html #}

    <!DOCTYPE html>
    <html lang="en">
        <head>
            {% assets "admin_js" %}
                <script type="text/javascript" src="{{ ASSET_URL }}"></script>
            {% endassets %}
            {% assets "admin_css" %}
                <link rel="stylesheet" href="{{ ASSET_URL }}" />
            {% endassets %}
        </head>
        <body>
        {% block body %}
        {% endblock %}
        </body>
    </html>

我们可以按照上面的方式在 “home” 部分的 *templates/home/layout.html* 中插入 “home” 束/包（bundles）。

使用过滤器
~~~~~~~~~~~~~

我们可以使用过滤器来预处理（pre-process）我们的静态文件。这是特别地方便用于压缩我们的 JavaScript 和 CSS 束/包（bundles）。

::

   # myapp/util/assets.py

   # [...]

   bundles = {

       'home_js': Bundle(
           'lib/jquery-1.10.2.js',
           'js/home.js',
           output='gen/home.js',
           filters='jsmin'),

       'home_css': Bundle(
           'lib/reset.css',
           'css/common.css',
           'css/home.css',
           output='gen/home.css',
           filters='cssmin'),

       'admin_js': Bundle(
           'lib/jquery-1.10.2.js',
           'lib/Chart.js',
           'js/admin.js',
           output='gen/admin.js',
           filters='jsmin'),

       'admin_css': Bundle(
           'lib/reset.css',
           'css/common.css',
           'css/admin.css',
           output='gen/admin.css',
           filters='cssmin')
   }

   # [...]

.. note::

    要使用 ``jsmin`` 和 ``cssmin`` 过滤器，你需要安装 ``jsmin`` 和 ``cssmin`` 包（例如，使用 ``pip install jsmin cssmin``）。确保也在 *requirements.txt* 中添加它们。

Flask-Assets 在模板第一次被渲染的时候会合并和压缩我们的文件，并且在其中一个源文件发生变化的时候会自动地更新压缩文件。

.. note::

   如果在你的配置中设置 `ASSETS_DEBUG = True`，Flask-Assets 将会输出每一个源文件而不是合并它们。

.. note::

   看看我们在 Flask-Assets 中可以使用的一些 `其它的过滤器 <http://elsdoerfer.name/docs/webassets/builtin_filters.html#js-css-compilers>`_。

摘要
-------

-  静态文件在 *static/* 目录里。
-  从你们自己的静态文件中分离出第三方库。
-  在模板中指定你的 favicon 位置。
-  使用 Flask-Assets 在你的模板中插入静态文件。
-  Flask-Assets 可以编译，合并和压缩你的静态文件。

.. [1]
    我们可以在 `源码 <https://github.com/miracle2k/webassets/blob/0.8/src/webassets/env.py#L380>`_ 中看看如何进行捆绑注册。
