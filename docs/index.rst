.. Flask-Migrate documentation master file, created by
   sphinx-quickstart on Fri Jul 26 14:48:13 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

欢迎访问Flask-Migrate的文档!
==========================================

**Flask-Migrate** 是一个扩展，它处理使用Alembic的Flask应用程序的SQLAlchemy数据库迁移。数据库操作可以通过Flask命令行界面或Flask-Script扩展来实现。

为什么要直接使用Flask-Migrate和Alembic？
--------------------------------------

Flask-Migrate 是一种扩展，它以正确的方式配置Alembic，以便使用Flask和Flask-SQLAlchemy应用程序。就实际的数据库迁移而言，一切都是由Alembic处理的，因此你可以获得完全相同的功能。

安装
------------

使用 `pip` 安装 Flask-Migrate::

    pip install Flask-Migrate

范例
-------

这是一个示例应用程序，它通过Flask-Migrate处理数据库迁移::

    from flask import Flask
    from flask_sqlalchemy import SQLAlchemy
    from flask_migrate import Migrate

    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

    db = SQLAlchemy(app)
    migrate = Migrate(app, db)

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(128))

使用上面的应用程序，你可以使用以下命令创建迁移存储库::

    $ flask db init

这将为你的应用程序添加一个 `migrations` 文件夹。这个文件夹的内容需要与其他源文件一起添加到版本控制中

然后你可以生成初始的迁移::

    $ flask db migrate
    
迁移脚本需要被检查和编辑, 因为Alembic目前还没有检测到你对模型所做的每一个更改。 特别是, Alembic目前无法检测表名更改、列名更改或匿名命名的约束。 A有关限制的详细摘要可在 `Alembic autogenerate documentation <http://alembic.zzzcomputing.com/en/latest/autogenerate.html#what-does-autogenerate-detect-and-what-does-it-not-detect>`_ 找到. 一旦完成，迁移脚本还需要添加到版本控制中。

然后你可以将迁移应用到数据库::

    $ flask db upgrade
    
然后，每次数据库模型更改时，重复 ``migrate`` 和 ``upgrade`` 命令。

要在另一个系统中同步数据库，只需刷新源代码控制中的 `migrations` 文件夹并运行 ``upgrade`` 命令.

要查看所有可用的命令，请运行此命令::

    $ flask db --help

请注意，应用程序脚本必须在 ``FLASK_APP`` 环境变量中设置，以便所有上述命令都能工作，这与 ``flask`` 命令行脚本所要求的一样。

使用 Flask-Script
------------------

Flask-Migrate 还支持Flask-Script命令行界面。这是一个示例应用程序，它通过Flask-Script公开所有数据库迁移命令::

    from flask import Flask
    from flask_sqlalchemy import SQLAlchemy
    from flask_script import Manager
    from flask_migrate import Migrate, MigrateCommand

    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

    db = SQLAlchemy(app)
    migrate = Migrate(app, db)

    manager = Manager(app)
    manager.add_command('db', MigrateCommand)

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(128))

    if __name__ == '__main__':
        manager.run()

假设上面的脚本存储在一个名为 ``manage.py``的文件中。运行脚本可以访问所有数据库迁移命令::

    $ python manage.py db init
    $ python manage.py db migrate
    $ python manage.py db upgrade
    $ python manage.py db --help

配置回调
-----------------------

有时应用程序需要动态地将自己的设置插入到Alembic配置中。用 ``configure`` 回调修饰的函数将在读取配置之后和使用配置之前被调用。该函数可以修改配置对象，或者用另一个对象替换它。

::

    @migrate.configure
    def configure_alembic(config):
        # modify config object
        return config

多个配置回调可以简单地通过修饰多个函数来定义。调用多个回调的顺序尚未确定。

多数据库支持
-------------------------

Flask-Migrate 可以与  `绑定 <http://flask-sqlalchemy.pocoo.org/binds/>`_  Flask-SQLAlchemy 的特性集成, 从而可以跟踪到与应用程序关联的多个数据库的迁移。

若要创建多个数据库迁移存储库，请添加 ``--multidb`` 参数到 ``init`` 命令::

    $ flask db init --multidb

使用这个命令，迁移存储库将被设置为跟踪主数据库上的迁移，以及在 ``SQLALCHEMY_BINDS`` 配置选项中定义的任何其他数据库上的迁移。

命令参考
-----------------

Flask-Migrate 公开了两个类, ``Migrate`` 和 ``MigrateCommand``. ``Migrate`` 类包含扩展的所有功能。 The ``MigrateCommand`` 类仅在需要通过 Flask-Script 扩展公开数据库迁移命令时才使用。

下面的示例使用标准的Flask命令行接口初始化扩展::

    from flask_migrate import Migrate
    migrate = Migrate(app, db)

``Migrate`` 的两个参数是应用程序实例和 Flask-SQLAlchemy 数据库实例。 ``Migrate`` 构造函数还接受其他关键字参数, 这些参数被传递给 Alembic 的 ``EnvironmentContext.configure()`` 方法。 作为所有 Flask 扩展的标准, 也可以使用 ``init_app`` 方法初始化Flask-Migrate 。

当使用 Flask-Script 的命令行界面时，扩展的初始化如下所示::

    from flask_migrate import Migrate, MigrateCommand
    migrate = Migrate(app, db)
    manager.add_command('db', MigrateCommand)

初始化扩展之后，将在命令行选项中添加一个 ``db`` 组，其中包含几个子命令，它们都位于 ``flask`` 命令中，或者使用 Flask-Script 创建一个 ``manage.py`` 类型的脚本。以下是可用子命令的列表:

- ``flask db --help``
    Shows a list of available commands.
    
- ``flask db init [--multidb]``
    初始化应用程序的迁移支持。 可选的 ``--multidb`` 使迁移对多个数据库配置为 `Flask-SQLAlchemy 绑定 <http://flask-sqlalchemy.pocoo.org/binds/>`_.
    
- ``flask db revision [--message MESSAGE] [--autogenerate] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]``
    创建一个空的修订脚本。 脚本需要通过升级和降级更改手动编辑。 查看 `Alembic 的文档 <http://alembic.zzzcomputing.com/en/latest/index.html>`_ 有关如何编写迁移脚本的说明。可以包含一个可选的迁移消息。
    
- ``flask db migrate [--message MESSAGE] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]``
    与 ``revision --autogenerate``相同。 迁移脚本中填充了自动检测到的更改。应该检查和编辑生成的脚本，因为并不是所有类型的更改都可以自动检测到。此命令不对数据库做任何更改，只创建修订脚本。

- ``flask db edit <revision>``
    使用$EDITOR编辑修订脚本。

- ``flask db upgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>``
    升级数据库。如果没有给出``revision`` ，则假定为``"head"``。
    
- ``flask db downgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>``
    降级数据库。如果没有给出``revision`` ，则假定为 ``-1``。
    
- ``flask db stamp [--sql] [--tag TAG] <revision>``
    将数据库中的修订设置为作为参数给出的修订，而不执行任何迁移。
    
- ``flask db current [--verbose]``
    显示数据库的当前修订。
    
- ``flask db history [--rev-range REV_RANGE] [--verbose]``
    显示迁移列表。如果没有给出范围，则显示整个历史。

- ``flask db show <revision>``
    显示由给定符号表示的修订。

- ``flask db merge [--message MESSAGE] [--branch-label BRANCH_LABEL] [--rev-id REV_ID] <revisions>``
    合并两个修订。创建一个新的修订文件。

- ``flask db heads [--verbose] [--resolve-dependencies]``
    在修订脚本目录中显示当前可用的头。

- ``flask db branches [--verbose]``
    显示当前分支点。

备注:
 
- 所有命令还接受一个 ``--directory DIRECTORY`` 选项，该选项指向包含迁移脚本的目录。如果省略该参数，则使用的目录是 ``migrations`` 。
- 默认目录也可以指定为 ``Migrate`` 构造函数的 ``directory`` 参数。
- 几个命令中的 ``--sql`` 选项执行 'offline' 模式迁移。 不执行数据库命令，而是将需要执行的SQL语句打印到控制台。
- 有关这些命令的详细文档可以在 `Alembic's command reference page <http://alembic.zzzcomputing.com/en/latest/api/commands.html>`_ 中找到。

API 参考
-------------

通过从模块 ``flask_migrate`` 导入函数，还可以通过编程方式访问 Flask-Migrate的命令行界面公开的命令。可用的功能如下:

- ``init(directory='migrations', multidb=False)``
    初始化应用程序的迁移支持。

- ``revision(directory='migrations', message=None, autogenerate=False, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)``
    创建一个空的修订脚本。

- ``migrate(directory='migrations', message=None, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)``
    创建一个自动修订脚本。

- ``edit(directory='migrations', revision='head')``
    使用 $EDITOR 编辑修订脚本(单个或多个）。

- ``merge(directory='migrations', revisions='', message=None, branch_label=None, rev_id=None)``
    合并两个修订。创建一个新的迁移文件。

- ``upgrade(directory='migrations', revision='head', sql=False, tag=None)``
    升级数据库。

- ``downgrade(directory='migrations', revision='-1', sql=False, tag=None)``
    降级数据库。
    
- ``show(directory='migrations', revision='head')``
    显示由给定符号表示的修订。

- ``history(directory='migrations', rev_range=None, verbose=False)``
    显示迁移列表。如果没有给出范围，则显示整个历史。

- ``heads(directory='migrations', verbose=False, resolve_dependencies=False)``
    在脚本目录中显示当前可用的头。

- ``branches(directory='migrations', verbose=False)``
    显示当前分支点

- ``current(directory='migrations', verbose=False, head_only=False)``
    显示数据库的当前修订。
    
- ``stamp(directory='migrations', revision='head', sql=False, tag=None)``
    将数据库中的修订设置为作为参数给出的修订，而不执行任何迁移。

注意:为了获得更大的脚本灵活性，你还可以直接使用Alembic公开的API。
