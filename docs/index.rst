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
    
迁移脚本需要被检查和编辑, 因为Alembic目前还没有检测到你对模型所做的每一个更改。 特别是, Alembic目前无法检测表名更改、列名更改或匿名命名的约束。 A有关限制的详细摘要可在 `Alembic autogenerate documentation <http://alembic.zzzcomputing.com/en/latest/autogenerate.html#what-does-autogenerate-detect-and-what-does-it-not-detect>`_找到. 一旦完成，迁移脚本还需要添加到版本控制中。

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

Flask-Migrate 可以与  `binds <http://flask-sqlalchemy.pocoo.org/binds/>`_  Flask-SQLAlchemy 的特性集成, 从而可以跟踪到与应用程序关联的多个数据库的迁移。

若要创建多个数据库迁移存储库，请添加 ``--multidb`` 参数到 ``init`` 命令::

    $ flask db init --multidb

使用这个命令，迁移存储库将被设置为跟踪主数据库上的迁移，以及在 ``SQLALCHEMY_BINDS`` 配置选项中定义的任何其他数据库上的迁移。

命令参考
-----------------

Flask-Migrate exposes two classes, ``Migrate`` and ``MigrateCommand``. The ``Migrate`` class contains all the functionality of the extension. The ``MigrateCommand`` class is only used when it is desired to expose database migration commands through the Flask-Script extension.

The following example initializes the extension with the standard Flask command-line interface::

    from flask_migrate import Migrate
    migrate = Migrate(app, db)

The two arguments to ``Migrate`` are the application instance and the Flask-SQLAlchemy database instance. The ``Migrate`` constructor also takes additional keyword arguments, which are passed to Alembic's ``EnvironmentContext.configure()`` method. As is standard for all Flask extensions, Flask-Migrate can be initialized using the ``init_app`` method as well.

When using Flask-Script's command-line interface, the extension is initialized as follows::

    from flask_migrate import Migrate, MigrateCommand
    migrate = Migrate(app, db)
    manager.add_command('db', MigrateCommand)

After the extension is initialized, a ``db`` group will be added to the command-line options with several sub-commands, both in the ``flask`` command or with a ``manage.py`` type script created with Flask-Script. Below is a list of the available sub-commands:

- ``flask db --help``
    Shows a list of available commands.
    
- ``flask db init [--multidb]``
    Initializes migration support for the application. The optional ``--multidb`` enables migrations for multiple databases configured as `Flask-SQLAlchemy binds <http://flask-sqlalchemy.pocoo.org/binds/>`_.
    
- ``flask db revision [--message MESSAGE] [--autogenerate] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]``
    Creates an empty revision script. The script needs to be edited manually with the upgrade and downgrade changes. See `Alembic's documentation <http://alembic.zzzcomputing.com/en/latest/index.html>`_ for instructions on how to write migration scripts. An optional migration message can be included.
    
- ``flask db migrate [--message MESSAGE] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]``
    Equivalent to ``revision --autogenerate``. The migration script is populated with changes detected automatically. The generated script should to be reviewed and edited as not all types of changes can be detected automatically. This command does not make any changes to the database, just creates the revision script.

- ``flask db edit <revision>``
    Edit a revision script using $EDITOR.

- ``flask db upgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>``
    Upgrades the database. If ``revision`` isn't given then ``"head"`` is assumed.
    
- ``flask db downgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>``
    Downgrades the database. If ``revision`` isn't given then ``-1`` is assumed.
    
- ``flask db stamp [--sql] [--tag TAG] <revision>``
    Sets the revision in the database to the one given as an argument, without performing any migrations.
    
- ``flask db current [--verbose]``
    Shows the current revision of the database.
    
- ``flask db history [--rev-range REV_RANGE] [--verbose]``
    Shows the list of migrations. If a range isn't given then the entire history is shown.

- ``flask db show <revision>``
    Show the revision denoted by the given symbol.

- ``flask db merge [--message MESSAGE] [--branch-label BRANCH_LABEL] [--rev-id REV_ID] <revisions>``
    Merge two revisions together. Creates a new revision file.

- ``flask db heads [--verbose] [--resolve-dependencies]``
    Show current available heads in the revision script directory.

- ``flask db branches [--verbose]``
    Show current branch points.

Notes:
 
- All commands also take a ``--directory DIRECTORY`` option that points to the directory containing the migration scripts. If this argument is omitted the directory used is ``migrations``.
- The default directory can also be specified as a ``directory`` argument to the ``Migrate`` constructor.
- The ``--sql`` option present in several commands performs an 'offline' mode migration. Instead of executing the database commands the SQL statements that need to be executed are printed to the console.
- Detailed documentation on these commands can be found in the `Alembic's command reference page <http://alembic.zzzcomputing.com/en/latest/api/commands.html>`_.

API Reference
-------------

The commands exposed by Flask-Migrate's command-line interface can also be accessed programmatically by importing the functions from module ``flask_migrate``. The available functions are:

- ``init(directory='migrations', multidb=False)``
    Initializes migration support for the application.

- ``revision(directory='migrations', message=None, autogenerate=False, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)``
    Creates an empty revision script.

- ``migrate(directory='migrations', message=None, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)``
    Creates an automatic revision script.

- ``edit(directory='migrations', revision='head')``
    Edit revision script(s) using $EDITOR.

- ``merge(directory='migrations', revisions='', message=None, branch_label=None, rev_id=None)``
    Merge two revisions together.  Creates a new migration file.

- ``upgrade(directory='migrations', revision='head', sql=False, tag=None)``
    Upgrades the database.

- ``downgrade(directory='migrations', revision='-1', sql=False, tag=None)``
    Downgrades the database.

- ``show(directory='migrations', revision='head')``
    Show the revision denoted by the given symbol.

- ``history(directory='migrations', rev_range=None, verbose=False)``
    Shows the list of migrations. If a range isn't given then the entire history is shown.

- ``heads(directory='migrations', verbose=False, resolve_dependencies=False)``
    Show current available heads in the script directory.

- ``branches(directory='migrations', verbose=False)``
    Show current branch points

- ``current(directory='migrations', verbose=False, head_only=False)``
    Shows the current revision of the database.
    
- ``stamp(directory='migrations', revision='head', sql=False, tag=None)``
    Sets the revision in the database to the one given as an argument, without performing any migrations.

Note: For greater scripting flexibility you can also use the API exposed by Alembic directly.
