
.. highlight:: python
    :linenothreshold: 0

编码约定
==================

.. image:: _static/images/conventions.png
   :alt: Coding conventions
   :height: 100 pt

在 Python 社区有一些指导你格式化代码的约定。如果你使用 Python 进行了一段时间开发，那么你可能已经熟悉了一些约定。我会继续让事情简单些并且留下一些 URLs，如果以前你还没有碰过这些话题的话你能够在这里 URLs 中找到更多的信息。

让我们来个 PEP 动员！
-----------------------

**PEP** 是“Python 增强倡议”，这些倡议是被索引以及托管在 python.org。在索引中，PEPs 被分成了几类，包含 meta-PEPs，这是比起技术更具有信息性。另一方面，技术的 PEPs 经常地描述像改进 Python 内部的东东。

这里有几个比较常用的 PEPs，像 PEP 8 以及 PEP 257，它们的目的是指导我们编写代码的方式。PEP 8 包含编码风格的指南。PEP 257 包含文档字符串指南，普遍的可接受的编写代码文档的方法。

PEP 8: Python 代码的风格指南
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PEP 8 是官方的 Python 代码风格指南。我建议你们阅读它并且在你的 Flask 项目中（以及你所有其它的 Python 代码）使用建议规范。你的代码会更加平易近人的，当开始有越来越多的代码行的时候。PEP 8 中的建议都是关于代码更具有可读性的。另外，如果你的项目将是开源的话，潜在的贡献者可能会期待或者愉快地使用 PEP 8 风格编写代码。

一个特别重要的建议就是每个缩进使用 4 个空格。缩进不使用制表符（Tab）。如果你违反这个约定（惯例）的话再项目进行切换的时候会给其他的开发人员带来负担。在任何语言中诸如此类的矛盾是很痛苦，但是空格符在 Python 中尤为重要，因此在真正的制表符（Tab）和空格符之间切换的话会导致许多问题，这对于调试是一个很大的麻烦。

PEP 257: 文档字符串约定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PEP 257 涉及到另一个 Python 标准：**文档字符串**。你可以在 PEP 里阅读定义和建议，但是这里是一个示例，这个示例给你一个文档字符串看起来像什么的概念：

::

   def launch_rocket():
      """Main launch sequence director.

      Locks seatbelts, initiates radio and fires engines.
      """
      # [...]

这种文字字符串可以使用软件比如 Sphinx 生成 HTML，PDF 以及其它格式的文本文件。它们会让人更容易理解你的代码。

.. note::

   - `PEP 8 <http://legacy.python.org/dev/peps/pep-0008/>`_
   - `PEP 257 <http://legacy.python.org/dev/peps/pep-0257/>`_
   - `Sphinx <http://sphinx-doc.org/>`_，由 Flask 相同的团队创建的文档生成器

相对导入
----------------

在开发 Flask 应用的时候，相对导入可以使得生活变得更加美好一些。前提是很简单的。比方说，我们想从 *myapp/models.py* 模块中导入 ``User`` 模型。你可能会考虑使用应用程序的包名，比如，``myapp.models``。使用相对导入，你应该指明相对于源文件的目标模块的位置。要做到这一点，我们使用点符号（.），第一个点符号（.）表示当前目录，每一个随后的点符号（.）表示下一个父目录。下面示例展示了绝对导入和相对导入之间的不同。

::

   # myapp/views.py

   # An absolute import gives us the User model
   from myapp.models import User

   # A relative import does the same thing
   from .models import User

这种方法的优点就是软件包变得更加模块化。现在你可以重命名你的包并且在其它项目中不需要更新硬编码的导入声明而重用模块。

在我的研究中我偶遇了一条 Tweet，它能说明相对导入的好处。

   刚刚不得不重命名了我们整个包。花费 1 秒，包的相对导入真酷！

   --- `David Beazley, @dabeaz <https://twitter.com/dabeaz/status/372059407711887360>`_

.. note::

   你可以从 `PEP 328 <http://www.python.org/dev/peps/pep-0328/#guido-s-decision>`_ 这一章节中阅读一些关于相对导入的语法。

摘要
-------

-  尽量遵循 PEP 8 中规定的编码风格约定。
-  尝试用定义在 PEP 257 中的文档字符串来记录你的应用程序。
-  使用相对导入来导入你的应用程序的内部模块。

