# docString+Sphinx 生成接口文档

docString 编写可参考主流标准。



## 环境准备

```shell
pip install sphinx  # Sphinx 主要功能是使用 reStructuredText, 把许多文件组织成一份结构合理的文档
pip install sphinx_rtd_theme # Sphinx 第三方主题
```



## 新建 Sphinx 项目

- 创建文件夹 `docTest`，进入 `docTest`，创建 python 文件放置的目录 `python`、文档目录 `doc`，进入 `doc` 运行指令 `sphinx-quickstart`

```shell
mkdir docTest
cd docTest
mkdir python
mkdir doc
cd doc
sphinx-quickstart
```

- 接下来会出现一些配置信息

```shell
……
You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: y  # 是否使用 source 和 build 文件夹分离的方式存储 Sphinx 输出

The project name will occur in several places in the built documentation.
> Project name: docTest      # 项目名称
> Author name(s): shan       # 作者名称
> Project release []: 0.0.1  # 项目版本号

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]:     # 文档语言，回车默认即可

……
```

- 成功运行之后在 `docTest/doc/` 下生成如下内容

```shell
.
├── build           # 用来存放通过make html生成文档网页文件的目录
├── make.bat
├── Makefile
└── source          # 存放用于生成文档的源文件
    ├── conf.py     # 在此文件修改项目的配置信息
    ├── index.rst   # 主文档
    ├── _static
    └── _templates
```

- 在 `docTest/doc/source/conf.py` 中修改配置：

```python
import os
import sys
sys.path.insert(0, os.path.abspath('../../python/'))   # python 文件位置，确保python源代码所在的包在系统路径中可以找到

html_theme = 'sphinx_rtd_theme'   # html 风格

extensions = ['sphinx.ext.autodoc']  
```



## 新建 Python 文件

为了展示文件和文件夹的链接方式，在 `docTest/python` 分别新建文件和文件夹

- 新建文件夹 `docTest/python/util/`，在里面放上一个文件 `initializers.py`

- 新建文件 `docTest/python/orthogonal.py`



## 关联 Sphnix 与 Python 文件

主要使用 [rst文件](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)。在 Sphnix 中，每一个 rst 代表一个单独的 html 页面，一般以 index.rst 作为主文件

- util.rst：链接到 util 文件夹。由于 initializer.py 中 calculate_gain 是方法，所以此处用 `automodule` 进行寻址。如果添加的是 class，使用 `autoclass`

```rst
Util
============================================================
.. currentmodule:: util.initializers
.. automodule:: util.initializers
    :members: 
        calculate_gain
    :member-order: bysource
```

- index.rst：通过不同的方式添加网页和内容

```rst
Welcome to demo's API Documentation
===================================
 
.. toctree::
   :maxdepth: 2
   :caption: Contents:
 
Folder in Python
======================
.. toctree::
   :maxdepth: 4

   util


File in Python
=======================
.. automodule:: orthogonal
   :members:

```



## 编译

- 在 `docTest/doc` 文件下执行如下代码，即能在 ``docTest/doc/build/html` 文件夹下获得编译好的网页

```shell
make html
```

