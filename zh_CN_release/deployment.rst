
.. highlight:: python
    :linenothreshold: 0

部署
==========

.. image:: _static/images/deployment.png
   :alt: Deployment

我们终于准备好向全世界展示我们的应用程序了。是时候要部署。这个过程可能是痛苦的因为有许多琐碎的事情要去做。当涉及到生产环境的搭建以及服务器的配置方案，这是有很多的选择需要做出。在本章中，我们会讨论一些重要的部分以及一些我们可能会用到的选项（关于主机或者服务器的搭建方式等等）。


主机
--------

首先我们需要一台服务器。现在有成千上万的服务器供应商，但是我个人建议的有三家。我不打算在这里介绍如何开始使用它们的细节，因为这是超出了本书的范围。相反我会讨论在它们上托管 Flask 应用程序的好处。

亚马逊网络服务 EC2
~~~~~~~~~~~~~~~~~~~~~~~

亚马逊网络服务是一个由亚马逊提供的服务的集合。有可能在之前你已经听说过它们可能成为新的创业者这些天的一个流行的选择。在亚马逊网络服务（AWS）中，我们最关心的是亚马逊弹性计算云（EC2）。亚马逊弹性计算云（EC2）最大的卖点就是我们得到一个虚拟服务器 - 或者用亚马逊网络服务（AWS）的说法叫做 **实例** - 几秒就能运行起来。如果我们迅速扩展我们的应用程序，只需要为我们的应用程序启动几个亚马逊弹性计算云（EC2）的实例并且把它们置于一个负载匀衡器之后（甚至我们能使用亚马逊网络服务（AWS）弹性负载均衡）。

至于 Flask，亚马逊网络服务（AWS）只是一个普通的虚拟服务器。我们可以在其上安装一个 linux 发行版并且安装我们的 Flask 应用程序以及我们服务器套件，无需太多的开销。这就意味着我们只需要一定的系统管理知识。

Heroku
~~~~~~

Heroku 是一个建立在亚马逊网络服务（AWS）之上的应用程序托管服务，像亚马逊弹性计算云（EC2）一样。它们让我们充分利用亚马逊弹性计算云（EC2）的便利性而且不需要系统管理的经验。

在 Heroku 中，我们使用 ``git push`` 部署我们的应用程序到它们的服务器上。这真的是很便利，我们不需要花费我们的时间在 SSHing 到一个服务器上，安装以及配置软件并且想出一个合理的部署过程。

这种便利性带来的就是价格问题，尽管亚马逊网络服务（AWS）和 Heroku 提供了一定的免费服务。

.. note::

   Heroku 有一个使用它们的服务 `部署 Flask 的教程 <https://devcenter.heroku.com/articles/getting-started-with-python>`_。

.. note::

   管理你自己的数据库是非常耗时并且做起来需要一定的经验。通过亲自为自己的项目管理数据库是一种好的学习方式，但是有时候你想要通过委托外部专业人士来节省时间。

   无论是 Heroku 还是亚马逊网络服务（AWS）都有数据库管理的产品。我个人还没有亲自使用过它们，但是我听说它们还不赖。如果你要确保你的数据被保护和备份并且无需自己做事情的话，它们值得考虑。

   - `Heroku Postgres <https://www.heroku.com/postgres>`_
   - `Amazon RDS <https://aws.amazon.com/rds/>`_

Digital Ocean
~~~~~~~~~~~~~

Digital Ocean 是亚马逊弹性计算云（EC2）的一个竞争对手，最近开始飞速地发展。像亚马逊弹性计算云（EC2）一样，Digital Ocean 让我们很快地启动一个虚拟服务器 - 现在称为 **droplets** 。所有的 droplets 是在固态硬盘（SSDs）上运行。对于我来说最大的卖点就是一个接口，这个接口比起亚马逊网络服务（AWS）控制面板更加简单和更加容易使用。Digital Ocean 是我偏爱的主机提供商，我建议你们可以看看它。

Digital Ocean 上部署 Flask 的过程与在亚马逊弹性计算云（EC2）上几乎一样。我们可以在其上安装一个 linux 发行版并且安装我们的 Flask 应用程序以及我们服务器套件。

.. note::

   Digital Ocean 为 Kickstarter 针对 *探索 Flask* 的活动做出足够大的贡献。就像我前面推荐使用 Digital Ocean，我保证我的建议是来自作为 Digital Ocean 用户的亲身经历。如果我不喜欢它的话，我也不会推荐它作为首选。

部署方式
---------

本节将介绍一些软件，我们需要安装这些软件在我们的服务器用来服务我们的 Flask 应用程序展示给全世界。最基本的部署方式就是有一个前置服务器，该服务器反向代理请求到运行我们 Flask 应用程序的应用运行器。我们通常也会有一个数据库，因此我们同样多多少少会讨论到这些方式。


应用程序运行器
~~~~~~~~~~~~~~~~~~

当我们开发我们的应用程序的时候，我们用来本地运行 Flask 的服务器不擅长处理真实的请求。当我们实际上需要把我们的应用程序面向公众的话，我们要使用一个应用程序运行器来运行它，像 Gunicorn 一样。Gunicorn 可以像线程一样处理请求和处理一些复杂的事情。

要使用 Gunicorn，我们要用 Pip 在我们的虚拟环境中安装 ``gunicorn`` 包。运行我们的应用程序是一个简单的命令行。

::

    # app.py

    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def index():
            return "Hello World!"

一个简单的应用程序已经完成。现在，为了用 Gunicorn 来服务于它，我们简单地运行 ``gunicorn`` 命令行。

::

   (ourapp)$ gunicorn rocket:app
   2014-03-19 16:28:54 [62924] [INFO] Starting gunicorn 18.0
   2014-03-19 16:28:54 [62924] [INFO] Listening at: http://127.0.0.1:8000 (62924)
   2014-03-19 16:28:54 [62924] [INFO] Using worker: sync
   2014-03-19 16:28:54 [62927] [INFO] Booting worker with pid: 62927

这时候，当我们的浏览器访问 *http://127.0.0.1:8000* 的时候，我们应该看到 "Hello World!"。

为了在后台运行这个服务器（即：作为一个守护进程），我们可以给 Gunicorn 传入 ``-D`` 参数。在这种方式下，即使我们关闭当前的终端会话，它依然运行。

如果我们把 Gunicorn 作为一个守护进程的话，我们可能会很难找到进程当后面我们要停止服务器的时候。我们能够告诉 Gunicorn 把进程 ID 放入到一个文件中以便后面我们能够停止或者重启它，而无需搜索整个运行程序的列表。我们使用 ``-p <file>`` 选项来做到这一点。

::

   (ourapp)$ gunicorn rocket:app -p rocket.pid -D
   (ourapp)$ cat rocket.pid
   63101

要重启以及杀死服务器，我们可以分别运行 ``kill -HUP`` 和 ``kill``。

::

   (ourapp)$ kill -HUP `cat rocket.pid`
   (ourapp)$ kill `cat rocket.pid`

默认情况下，Gunicorn 运行在端口 8000。我们可以通过添加 ``-b`` 绑定选项来更改端口。

::

   (ourapp)$ gunicorn rocket:app -p rocket.pid -b 127.0.0.1:7999 -D

让 Gunicorn 对外开放
^^^^^^^^^^^^^^^^^^^^^^

.. warning::

   Gunicorn 是要在一个反向代理的后面。如果你要让它接收来自外部公众的请求，它很容易地遭受拒绝服务攻击。它很难处理这些类型的请求。因此仅仅允许外部的连接为调试的目的并且确保在实际运行中切回到只允许内部连接。

如果我们像上面介绍的运行 Gunicorn 的话，我们无法接收到外部的请求，只能接收到本机的请求。这是因为 Gunicorn 默认是绑定到 127.0.0.1。这就意味着它仅仅监听来自服务器本机的连接。这就是我们希望的运行方式，我们会有一个反向代理的服务器，它位于外部于我们的 Gunicorn 服务器之间。然而，如果我们需要为了调试目的接收来自外部的请求，我们可以要求 Gunicorn 绑定到 0.0.0.0。这就是告诉它监听所有的请求。

::

    (ourapp)$ gunicorn rocket:app -p rocket.pid -b 0.0.0.0:8000 -D

.. note::

   - `在官方文档 <http://docs.gunicorn.org/en/latest/>`_ 中阅读更多关于运行以及部署 Gunicorn 的内容。
   - `Fabric <http://docs.fabfile.org/en/latest>`_ 是一个工具，它让你舒适地在本机运行所有这些部署以及管理命令行，无需 SSHing 到每一台服务器。

Nginx 反向代理
~~~~~~~~~~~~~~~~~~~

一个反向代理处理公开的 HTTP 请求，发送它们到 Gunicorn 并且给出响应回到请求的客户端。Nginx 可以很有效地用于一个反向代理并且 Gunicorn “强烈建议” 我们使用 Nginx。

为了配置 Nginx 作为运行在 127.0.0.1:8000 上的 Gunicorn 服务器的一个反向代理，我们可以为我们的应用程序创建一个文件：
*/etc/nginx/sites-available/expl-oreflask.com*。

::

    # /etc/nginx/sites-available/exploreflask.com

    # Redirect www.exploreflask.com to exploreflask.com
    server {
            server_name www.exploreflask.com;
            rewrite ^ http://exploreflask.com/ permanent;
    }

    # Handle requests to exploreflask.com on port 80
    server {
            listen 80;
            server_name exploreflask.com;

                    # Handle all locations
            location / {
                            # Pass the request to Gunicorn
                    proxy_pass http://127.0.0.1:8000;
                    
                    # Set some HTTP headers so that our app knows where the 
                    # request really came from
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
    }

现在我们创建一个符号链接指向这个文件到 */etc/nginx/sites-enabled* 上并且重启 Nginx。

::

    $ sudo ln -s \
    /etc/nginx/sites-available/exploreflask.com \
    /etc/nginx/sites-enabled/exploreflask.com

我们现在应该可以让 Nginx 接收我们的请求并且接收来自我们的应用程序的响应。

.. note::

   在 Gunicorn 文档中的 `Nginx 配置部分 <http://docs.gunicorn.org/en/latest/deploy.html#nginx-configuration>`_ 会给你更多关于设置 Nginx 反向代理的信息。

ProxyFix
^^^^^^^^

我们可能会碰到使用 Flask 不能处理代理请求的一些问题。这是与我们设置 Nginx 中配置的那些头有关。我们可以使用 Werkzeug 的 ProxyFix 来修复代理请求的问题。

::

    # app.py

    from flask import Flask

    # Import the fixer
    from werkzeug.contrib.fixers import ProxyFix

    app = Flask(__name__)

    # Use the fixer
    app.wsgi_app = ProxyFix(app.wsgi_app)

    @app.route('/')
    def index():
            return "Hello World!"

.. note::

   - 在 `Werkzeug 官方文档 <http://werkzeug.pocoo.org/docs/contrib/fixers/#werkzeug.contrib.fixers.ProxyFix>`_ 中获取更多关于 ProxyFix 的信息。

摘要
-------

-  托管 Flask 应用程序的三个好的选择是 AWS EC2，Heroku 以及 Digital Ocean。
-  对于一个 Flask 应用程序最合理的部署搭配就是一个应用程序运行器和一个像 Nginx 的反向代理。
-  Gunicorn 应该位于 Nginx 后面并且监听在 127.0.0.1（内部请求）而不是监听在 0.0.0.0（外部请求）。
-  在你的 Flask 应用程序中使用 Werkzeug 的 ProxyFix 来处理合适的代理头。

