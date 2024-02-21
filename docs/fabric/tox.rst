**************
tox.ini 文件
**************

什么是 tox
===========

tox.ini 是用于配置和管理 Python 项目的 tox 工具的配置文件。tox 是一个用于自动化测试、构建和配置环境的工具，特别是用于在多个 Python 版本和环境中运行测试。

tox.ini 文件通常包含以下信息：

- 环境配置： 定义要测试的环境，包括 Python 版本、依赖项等。

- 测试命令： 定义运行测试的命令，通常是使用测试运行器（如 pytest、nose 等）运行测试套件。

- 其他配置选项： 包括其他 tox 相关的配置选项，如代码风格检查、覆盖率检查等。

.. code-block:: ini

    [tox]
    envlist = py36, py37, py38

    [testenv]
    deps =
        pytest
    commands =
        pytest tests/

上述配置指定了三个测试环境（Python 3.6、Python 3.7 和 Python 3.8），并定义了使用 pytest 运行位于 tests/ 目录下的测试套件的命令。[tox] 部分包含全局配置，而 [testenv] 部分包含每个测试环境的配置。

通过运行 ``tox`` 命令，tox 将按照配置在不同的环境中运行测试，确保项目在各种环境中都能正常工作。

内容介绍
=========

.. code-block:: ini

    [tox]
    minversion = 3.4
    envlist =
        docs,
        docs-linkcheck
    skipsdist=true

此部分定义了全局配置。

- minversion = 3.4: 指定了 tox 的最低版本要求。
- envlist: 列出了定义的测试环境，以及要在每个环境中运行的任务。

.. code-block:: ini

    [testenv:docs]
    deps = -rdocs/requirements.txt
    commands =
        sphinx-build -b html -n -d {envtmpdir}/doctrees ./docs/source {toxinidir}/docs/build/html
        echo "Generated docs available in {toxinidir}/docs/build/html"
    whitelist_externals = echo
    basepython=python3.7
    ignore_basepython_conflict=True

定义了一个名为 docs 的测试环境。

- deps = -rdocs/requirements.txt: 指定了 docs 环境所需的依赖项，从 docs/requirements.txt 文件中读取。
- commands: 定义了在此环境中运行的命令。在这里，它使用 Sphinx 构建工具生成 HTML 格式的文档。
- whitelist_externals = echo: 允许使用外部命令 echo。
- basepython=python3.7: 指定了在此环境中使用的 Python 版本。
- ignore_basepython_conflict=True: 忽略 Python 版本冲突。

.. code-block:: ini

    [testenv:docs-linkcheck]
    deps = -rdocs/requirements.txt
    commands =
        sphinx-build -b linkcheck -d {envtmpdir}/doctrees ./docs/source {toxinidir}/docs/build/linkcheck
    basepython=python3.7
    ignore_basepython_conflict=True

定义了一个名为 docs-linkcheck 的测试环境，与上述环境类似，这里是用于检查文档中的链接是否有效。
