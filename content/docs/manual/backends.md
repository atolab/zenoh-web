---
title: "Zenoh backends and storages"
weight : 2200
menu:
  docs:
    parent: manual
---

In zenoh a *backend* is a storage technology. 
Concretely, it's a software library which provides implementation of [Storages](../abstractions#storage). It usually leverages a third-party technology (e.g. InfluxDB, SQLite, PostgreSQL...) to store the keys/values published in zenoh.

The [Storages plugin](../plugin-storages) manages the backends and storages for each zenoh router.

---------
## Backends management

You can manage the backends from a zenoh router via the [admin space](../abstractions#admin-space) using zenoh PUT/GET/DELETE operations on the Path:  
**`/@/router/<router-id>/plugin/storages/backend/<backend-id>`**  
where **`<backend-id>`** is a free identifier for the backend (must be unique per-router).

### Add a Backend

To add a Backend, put a Properties Value on the admin path for backend (assuming it doesn't exist yet).  
The set of properties depend on the backend implementation. They all accept a `"lib"` property to specify the backend library file to load. If it is not specified, the [Storages plugin](../plugin-storages) searches for a library with the name: `"zbackend_<backend-id>"`.

The following is an example of REST API to add an InfluxDB backend (using the `zbackend_influxdb` library):  
```bash
curl -X PUT -H 'content-type:application/properties' -d "url=http://localhost:8086" http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
```

This is a similar example with the Python API, specifying the location of the backend library:  
```python
from zenoh import Zenoh, Value

z = Zenoh({"mode": "client"})
workspace = z.workspace()
workspace.put('/@/router/local/plugin/storages/backend/influxdb',
    {'url': 'http://localhost:8086', 'lib': '/opt/zenoh/lib/libzbackend_influxdb.so'})
```

### Backend status

To see the status of the Backend, you can do a GET on the admin path.  
The resulting Value depends on the Backend implementation, but is usually in JSON format.

Example with REST API:
```bash
curl http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
```

### Remove a Backend

To remove a Backend (and all its Storages), delete its admin space:

Example with REST API:
```bash
curl -X DELETE http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
```


-----------
## Storages management

Each Storage belongs to a Backend. You can manage a Storage via the [admin space](../abstractions#admin-space) using zenoh PUT/GET/DELETE operations on the Path:  
**`/@/router/<router-id>/plugin/storages/backend/<backend-id>/storage/<storage-id>`**  
where **`<backend-id>`** is the identifier of the Backend hosting the Storage, and **`<storage-id>`** is a free identifier for the storage (must be unique per-backend).

### Add a Storage

To add a Storage, put a Properties Value on the admin path (assuming it doesn't exist yet).  
The set of properties to use depends on the backend implementation. They all require a `"path_expr"` property to specify the [Path Expression](../abstractions#path-expression) the Storage subscribes to (to store PUT keys/values) and replies on (for GET operations).

Examples with REST API:  
```bash
# for a memory storage
curl -X PUT -H 'content-type:application/properties' -d "path_expr=/demo/example/**" http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/example

# for an InfluxDB storage
curl -X PUT -H 'content-type:application/properties' -d "path_expr=/demo/example/**;path_prefix=/demo/example;on_closure=drop_series;db=example;create_db" http://localhost:8000/@/router/local/plugin/storages/backend/influxdb/storage/example
```

Similar examples with Python API:  
```python
# for a memory storage
workspace.put('/@/router/local/plugin/storages/backend/memory/storage/example',
    {'path_expr': '/demo/example/**'})

# for an InfluxDB storage
workspace.put('/@/router/local/plugin/storages/backend/influxdb/storage/example',
    {'path_expr': '/demo/example/**', 'path_prefix': '/demo/example', 'on_closure': 'drop_series', 'db': 'example', 'create_db': 'true'})
```


### Storage status

To see the status of the Storage, you can do a GET on the admin path.  
The resulting Value depends on the Backend implementation, but is usually in JSON format.

Example with REST API:
```bash
curl http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/example
```

### Remove a Storage

To remove a Storage, delete its admin space:

Example with  REST API:
```bash
curl -X DELETE http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/example
```

<--

The supported backends and their expected properties are:

 - at **initialization**. I.e. the properties to give to the [Admin space](../abstractions#admin-space) when you add the backend.
 - at **storage creation**. I.e. the properties to give to the [Admin space](../abstractions#admin-space) when you add the storage.

## Memory

A backend which uses zenoh main memory is not persistent, as soon as the zenoh router stops, all the keys/values stored in this backend are lost.

- **plugin file**: *none - the memory backend is built-in with zenoh*
- **initialization properties**: *none*
- **storage creation properties**:
  - **selector**: the storage's [selector](../abstractions#selector)

Example of a memory storage creation using REST API:
```bash
curl -X PUT -d '{"selector":"/demo/test/**"}' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-test
```

---
## SQLite 3

A backend which uses a [SQLite 3](https://www.sqlite.org) database. Each storage maps to an SQL table.

- **plugin file**: zenoh-storages-be-sqlite3.cmxs
- **initialization properties**:
  - **lib**=sqlite3
  - **url**: a URL pointing to a SQLite3 database file. Syntax: **`sqlite3://</path/to/file>`**
- **storage creation properties**:
  - **selector**: the storage's [selector](../abstractions#selector)
  - **table**: *optional* - the name of the SQL table to use for storage. If not specified, zenoh creates a new table. The schema of the table is:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size**: *optional* - the size of the `k` VARCHAR that zenoh uses for table creation. Default value: 3072
  - **on_dispose**: *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop**: the table is dropped (i.e. removed)
     - **truncate**: the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset*: the table remains untouched (this is the default behaviour)

Example of the SQLite3 backend creation using REST API:
```bash
curl -X PUT -d '{"lib":"sqlite3","url":"sqlite3:///tmp/sqlite3-zenoh.db"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3
```
Example of a SQLite3 storage creation using REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3/storage/mytable
```

---
## PostgreSQL

A backend which uses a [PostgreSQL](https://www.postgresql.org) database. Each storage maps to an SQL table.

- **plugin file**: zenoh-storages-be-postgresql.cmxs
- **initialization properties**:
  - **lib**=postgresql
  - **url**: a PostgreSQL connection URL. Syntax: **`postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]`**
- **storage creation properties**:
  - **selector**: the storage's [selector](../abstractions#selector)
  - **table**: *optional* - the name of the SQL table to use for storage. If not specified, zenoh creates a new table. The schema of the table is:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size**: *optional* - the size of the `k` VARCHAR that zenoh uses for table creation. Default value: 3072
  - **on_dispose**: *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop**: the table is dropped (i.e. removed)
     - **truncate**: the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset*: the table remains untouched (this is the default behaviour)

Example of the PostgreSQL backend creation using REST API:
```bash
curl -X PUT -d '{"lib":"postgresql","url":"postgresql://localhost/mydb"}' http://localhost:8000/@/router/local/plugin/storages/backend/psql
```
Example of a PostgreSQL storage creation using REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/psql/storage/mytable
```

---
## MariaDB

A backend which uses a [MariaDB](https://mariadb.org/) (or MySQL) database. Each storage maps to an SQL table.

- **plugin file**: zenoh-storages-be-mariadb.cmxs
- **initialization properties**:
  - **lib**=mariadb
  - **url**: a MariaDB connection URL. Syntax: **`mariadb://[user[:password]@][host][:port][/dbname][?param1=value1&...]`**
- **storage creation properties**:
  - **selector**: the storage's [selector](../abstractions#selector)
  - **table**: *optional* - the name of the SQL table to use for storage. If not specified, zenoh creates a new table. The schema of the table is:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size**: *optional* - the size of the `k` VARCHAR that zenoh uses for table creation. Default value: 3072
  - **on_dispose**: *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop**: the table is dropped (i.e. removed)
     - **truncate**: the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset*: the table remains untouched (this is the default behaviour)

Example of the MariaDB backend creation using REST API:
```bash
curl -X PUT -d '{"lib":"mariadb","url":"mariadb://localhost/mydb"}' http://localhost:8000/@/router/local/plugin/storages/backend/mariadb
```
Example of a MariaDB storage creation using REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/mariadb/storage/mytable
```

---
## InfluxDB

A backend which uses an [InfluxDB](https://docs.influxdata.com/influxdb) service. Each storage maps to a database. Each zenoh path to store, maps to an InfluxDB [measurement](https://docs.influxdata.com/influxdb/v1.7/concepts/key_concepts/#measurement) named with the relative path from the storage's selector. Each key/value put into the storage, maps to InfluxDB [point](https://docs.influxdata.com/influxdb/v1.7/concepts/key_concepts/#point) reusing the timestamp set by zenoh
(but with a precision of nanoseconds). The encoding and the original zenoh timestamp are used as tags within the point.

- **plugin file**: zenoh-storages-be-influxdb.cmxs
- **initialization properties**:
  - **lib**=influxdb
  - **url**: an InfluxDB HTTP URL. Syntax: **`http://<host>:<port>`**
- **storage creation properties**:
  - **selector**: the storage's [selector](../abstractions#selector)
  - **db**: *optional* - the name of the database to use for this storage. If not specified, zenoh creates a new database.
  - **on_dispose**: *optional* - the strategy to use when the storage is removed. There are 4 options:
     - **drop** or **DropDB**: the database is dropped (i.e. removed)
     - **DropAllSeries**: all the series (measurements) are dropped (i.e. data are deleted, but the database remains empty)
     - *unset*: the database remains untouched (this is the default behaviour)

Example of the InfluxDB backend creation using REST API:
```bash
curl -X PUT -d '{"lib":"influxdb","url":"http://localhost:8086"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
```
Example of a InfluxDB storage creation using REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/influxdb/**","db":"mydb","on_dispose":"DropAllSeries"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb/storage/mydb
```

By default a `get` matching an InfluxDB storage returns the last value of each measurement with a key that matches the `get`selector.  
But you can also get a series of values for each matching key using the `starttime` and `stoptime` properties in the selector. Those properties support the InfluxDB **[time syntax](https://docs.influxdata.com/influxdb/v1.7/query_language/data_exploration/#time-syntax)** (the part after the operator).
For instance:

   - `/demo/example/influxdb/**?(starttime=2019-01-01)`
   - `/demo/example/influxdb/**?(starttime=2019-11-01T09:30:00.000000000Z;stoptime=now())`
   - `/demo/example/influxdb/**?(starttime=now()-2d;stoptime=now()-1d)`
-->