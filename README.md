bottle-pgsql
============
This plugin is based on [bottle-mysql](https://github.com/MTecknology/bottle-mysql) 
and [bottle-sqlite](https://github.com/bottlepy/bottle-extras/tree/master/sqlite).

This plugin simplifies the use of postgresql databases in your Bottle applications.
Once installed, all you have to do is to add a ``db`` keyword argument
(configurable) to route callbacks that need a database connection.

Installation
------------
Install with one of the following commands:

    $ pip install bottle-pgsql
    $ easy_install bottle-pgsql

or download the latest version from github:

    $ git clone https://github.com/raisoblast/bottle-pgsql.git
    $ cd bottle-pgsql
    $ python setup.py install

Usage
-----
Once installed to an application, the plugin passes an open
PostgreSQL connection instance to all routes that require a ``db`` keyword
argument::
```python
    import bottle
    import bottle_pgsql

    app = bottle.Bottle()
    plugin = bottle_pgsql.Plugin('dbname=db user=user password=pass')
    app.install(plugin)

    @app.route('/show/:item')
    def show(item, db):
        db.execute('SELECT * from items where name="%s"', (item,))
        row = db.fetchone()
        if row:
            return template('showitem', page=row)
        return HTTPError(404, "Entity not found")
```

Routes that do not expect a ``db`` keyword argument are not affected.

The connection handle is configured so that `sqlite3.Row` objects can be
accessed both by index (like tuples) and case-insensitively by name. At the end of
the request cycle, outstanding transactions are committed and the connection is
closed automatically. If an error occurs, any changes to the database since the
last commit are rolled back to keep the database in a consistent state.

Configuration
-------------
The following configuration options exist for the plugin class:

* **dsn**: data source name (default: empty)
* **keyword**: The keyword argument name that triggers the plugin (default: 'db').
* **autocommit**: Whether or not to commit outstanding transactions at the end of the request cycle (default: True).
* **dictrows**: Whether or not to support dict-like access to row objects (default: True).

You can override each of these values on a per-route basis::
```python
    @app.route('/cache/:item', pgsql={'dsn': 'dbname=otherdb'})
    def cache(item, db):
        ...
```   
or install two plugins with different ``keyword`` settings to the same application:
```python
    app = bottle.Bottle()
    test_db = bottle_pgsql.Plugin('dbname=db_test')
    cache_db = bottle_pgsql.Plugin('dbname=db_cache')
    app.install(test_db)
    app.install(cache_db)

    @app.route('/show/:item')
    def show(item, db):
        ...

    @app.route('/cache/:item')
    def cache(item, cache):
        ...
```
