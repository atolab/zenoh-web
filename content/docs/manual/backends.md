---
title: "Zenoh backends"
weight : 2020
menu:
  docs:
    parent: manual
---

Here is the list of the supported backends and their expected properties:

 - at **initialization**. I.e. the properties to give to the [Admin space](../abstractions#admin-space) when adding the backend.
 - at **storage creation**. I.e. the properties to give to the [Admin space](../abstractions#admin-space) when adding the storage.

## Memory

A backend using zenoh main memory.
It's not persistent: as soon as the zenoh router stops, all the keys/values stored in this backend are lost.

- **plugin file**: *none - the memory backend is built-in with zenoh.*
- **initialization properties**: *none*
- **storage creation properties**:
  - **selector** : the storage's [selector](../abstractions#selector)

Example of a memory storage creation using the REST API:
```bash
curl -X PUT -d '{"selector":"/demo/test/**"}' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-test
```

---
## SQLite 3

A backend using a [SQLite 3](https://www.sqlite.org) database.  
Each storage will map to a SQL table.

- **plugin file**: zenoh-storages-be-sqlite3.cmxs
- **initialization properties**:
  - **lib**=sqlite3
  - **url** : an URL pointing to a SQLite3 database file. Syntax: **`sqlite3://</path/to/file>`**
- **storage creation properties**:
  - **selector** : the storage's [selector](../abstractions#selector)
  - **table** : *optional* - the name of the SQL table to use for storage.
    If not specified, zenoh will create a new table.
    The schema of the table will be:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size** : *optional* - the size of the `k` VARCHAR that zenoh will use for table creation. Default value: 3072.
  - **on_dispose** : *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop** : the table is dropped (i.e. removed)
     - **truncate** : the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset* : the table remains untouched (this is the default behaviour)

Example of the SQLite3 backend creation using the REST API:
```bash
curl -X PUT -d '{"lib":"sqlite3","url":"sqlite3:///tmp/sqlite3-zenoh.db"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3
```
Example of a SQLite3 storage creation using the REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3/storage/mytable
```

---
## PostgreSQL

A backend using a [PostgreSQL](https://www.postgresql.org) database.  
Each storage will map to a SQL table.

- **plugin file**: zenoh-storages-be-postgresql.cmxs
- **initialization properties**:
  - **lib**=postgresql
  - **url** : a PostgreSQL connection URL. Syntax: **`postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]`**
- **storage creation properties**:
  - **selector** : the storage's [selector](../abstractions#selector)
  - **table** : *optional* - the name of the SQL table to use for storage.
    If not specified, zenoh will create a new table.
    The schema of the table will be:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size** : *optional* - the size of the `k` VARCHAR that zenoh will use for table creation. Default value: 3072.
  - **on_dispose** : *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop** : the table is dropped (i.e. removed)
     - **truncate** : the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset* : the table remains untouched (this is the default behaviour)

Example of the PostgreSQL backend creation using the REST API:
```bash
curl -X PUT -d '{"lib":"postgresql","url":"postgresql://localhost/mydb"}' http://localhost:8000/@/router/local/plugin/storages/backend/psql
```
Example of a PostgreSQL storage creation using the REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/psql/storage/mytable
```

---
## MariaDB

A backend using a [MariaDB](https://mariadb.org/) (or MySQL) database.  
Each storage will map to a SQL table.

- **plugin file**: zenoh-storages-be-mariadb.cmxs
- **initialization properties**:
  - **lib**=mariadb
  - **url** : a MariaDB connection URL. Syntax: **`mariadb://[user[:password]@][host][:port][/dbname][?param1=value1&...]`**
- **storage creation properties**:
  - **selector** : the storage's [selector](../abstractions#selector)
  - **table** : *optional* - the name of the SQL table to use for storage.
    If not specified, zenoh will create a new table.
    The schema of the table will be:  
        `(k VARCHAR(3072) NOT NULL PRIMARY KEY, v TEXT, e VARCHAR(10), t VARCHAR(60))`  
    where `k` is the key (a path relative to the selector), `v` is the value as a string, `e` is the value encoding,
    and `t` is the timestamp.
  - **key_size** : *optional* - the size of the `k` VARCHAR that zenoh will use for table creation. Default value: 3072.
  - **on_dispose** : *optional* - the strategy to use when the storage is removed. There are 3 options:
     - **drop** : the table is dropped (i.e. removed)
     - **truncate** : the table is truncated (i.e. data are deleted, but the table remains empty)
     - *unset* : the table remains untouched (this is the default behaviour)

Example of the MariaDB backend creation using the REST API:
```bash
curl -X PUT -d '{"lib":"mariadb","url":"mariadb://localhost/mydb"}' http://localhost:8000/@/router/local/plugin/storages/backend/mariadb
```
Example of a MariaDB storage creation using the REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/sql/**","table":"mytable","key_size":"100","on_dispose":"truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/mariadb/storage/mytable
```

---
## InfluxDB

A backend using an [InfluxDB](https://docs.influxdata.com/influxdb) service.  
Each storage will map to a database.
Each zenoh path to store will map to a an InfluxDB 
[measurement](https://docs.influxdata.com/influxdb/v1.7/concepts/key_concepts/#measurement)
named with the relative path from the storage's selector.
Each key/value put into the storage will map to InfluxDB
[point](https://docs.influxdata.com/influxdb/v1.7/concepts/key_concepts/#point) reusing the timestamp set by zenoh
(but with a precision of nanoseconds). The encoding and the original zenoh timestamp are used as tags within the point.

- **plugin file**: zenoh-storages-be-influxdb.cmxs
- **initialization properties**:
  - **lib**=influxdb
  - **url** : an InfluxDB HTTP URL. Syntax: **`http://<host>:<port>`**
- **storage creation properties**:
  - **selector** : the storage's [selector](../abstractions#selector)
  - **db** : *optional* - the name of the database to use for this storage.
    If not specified, zenoh will create a new database.
  - **on_dispose** : *optional* - the strategy to use when the storage is removed. There are 4 options:
     - **drop** or **DropDB** : the database is dropped (i.e. removed)
     - **DropAllSeries** : all the series (measurements) are dropped (i.e. data are deleted, but the database remains empty)
     - *unset* : the database remains untouched (this is the default behaviour)

Example of the InfluxDB backend creation using the REST API:
```bash
curl -X PUT -d '{"lib":"influxdb","url":"http://localhost:8086"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
```
Example of a InfluxDB storage creation using the REST API:
```bash
curl -X PUT -d '{"selector":"/demo/example/influxdb/**","db":"mydb","on_dispose":"DropAllSeries"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb/storage/mydb
```

By default a `get` matching an InfluxDB storage will return the last value of each measurement with a key
that matches the `get`selector.  
But you can also get a serie of values for each matching key using the `starttime` and `stoptime` properties in the selector. Those properties support the InfluxDB **[time syntax](https://docs.influxdata.com/influxdb/v1.7/query_language/data_exploration/#time-syntax)** (the part after the operator).
For instance:

   - `/demo/example/influxdb/**?(starttime=2019-01-01)`
   - `/demo/example/influxdb/**?(starttime=2019-11-01T09:30:00.000000000Z;stoptime=now())`
   - `/demo/example/influxdb/**?(starttime=now()-2d;stoptime=now()-1d)`
