---
title: "InfluxDB backend"
weight : 2220
menu:
  docs:
    parent: manual
---


A backend using an [InfluxDB](https://docs.influxdata.com/influxdb) service to store keys/values as time-series.

 - **Library name**: `zbackend_influxdb`

## Behaviour

### Mapping to InfluxDB concepts
Each **storage** will map to an InfluxDB **database**.  
Each **path** to store will map to a an InfluxDB
[**measurement**](https://docs.influxdata.com/influxdb/v1.8/concepts/key_concepts/#measurement)
named with the path stripped from the `"path_prefix"` property (see below).  
Each **key/value** put into the storage will map to an InfluxDB
[**point**](https://docs.influxdata.com/influxdb/v1.8/concepts/key_concepts/#point) reusing the timestamp set by zenoh
(but with a precision of nanoseconds). The fileds and tags of the point is are the following:
 - `"kind"` tag: the zenoh change kind (`"PUT"` for a value that have been put, or `"DEL"` to mark the deletion of the path)
 - `"timestamp"` field: the original zenoh timestamp
 - `"encoding"` field: the value's encoding flag
 - `"base64"` field: a boolean indicating if the value is encoded in base64
 - `"value"`field: the value as a string, possibly encoded in base64 for binary values.

### Behaviour on deletion
On deletion of a path, all points with a timestamp before the deletion message are deleted.
A point with `"kind"="DEL`" is inserted (to avoid re-insertion of points with an older timestamp in case of un-ordered messages).
After a delay (5 seconds), the measurement corresponding to the deleted path is dropped if it still contains no points.

### Behaviour on GET
On GET operations, by default the storage returns only the latest point for each path/measurement.
This is to be coherent with other backends technologies that only store 1 value per-key.  
If you want to get time-series as a result of a GET operation, you need to specify the `"starttime"` and/or `"stoptime"`
properties in your [Selector](../abstractions#selector).

Examples of selectors:
```bash
  # get the complete time-series
  /demo/example/**?(starttime=0)

  # get points within a fixed date interval
  /demo/example/influxdb/**?(starttime=2020-01-01;starttime=2020-01-02T12:00:00.000000000Z)

  # get points within a relative date interval
  /demo/example/influxdb/**?(starttime=now()-2d;stoptime=now()-1d)
```

The `"starttime"` and `"stoptime"` properties support the InfluxDB **[time syntax](https://docs.influxdata.com/influxdb/v1.8/query_language/explore-data/#time-syntax)** (*<rfc3339_date_time_string>*, *<rfc3339_like_date_time_string>*, *<epoch_time>* and relative time using `now()`).

---------
## Properties for Backend creation

- **`"lib"`** (optional) : the backend library name. If not speficied, the Backend identifier in admin space must be `influxdb`.

- **`"url"`** (**required**) : an URL to the InfluxDB service. Example: `http://localhost:8086`

- **`"username"`** (optional) : an [InfluxDB admin](https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/#admin-users) user name. It will be used for creation of databases, granting read/write privileges of databases mapped to storages and dropping of databases and measurements.

- **`"password"`** (optional) : the admin user's password.


---------
## Properties for Storage creation

- **`"path_expr"`** (**required**) : the Storage's [Path Expression](../abstractions#path-expression)

- **`"path_prefix"`** (optional) : a prefix of the `"path_expr"` that will be stripped from each path to store.  
  _Example: with `"path_expr"="/demo/example/**"` and `"path_prefix"="/demo/example/"` the path `"/demo/example/foo/bar"` will be stored as key: `"foo/bar"`. But replying to a get on `"/demo/**"`, the key `"foo/bar"` will be transformed back to the original path (`"/demo/example/foo/bar"`)._

- **`"db"`** (optional) : the InfluxDB database name the storage will map into. If not specified, a random name will be generated, and the corresponding database will be created (even if `"create_db"` is not set).

- **`"create_db"`** (optional) : create the InfluxDB database if not already existing. *(the value doesn't matter, only the property existence is checked)*

- **`"on_closure"`** (optional) : the strategy to use when the Storage is removed. There are 3 options:
  - *unset*: the database remains untouched (this is the default behaviour)
  - `"drop_db"`: the database is dropped (i.e. removed)
  - `"drop_series"`: all the series (measurements) are dropped and the database remains empty.

- **`"username"`** (optional) : an InfluxDB user name (usually [non-admin](https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/#non-admin-users)). It will be used to read/write points in the database on GET/PUT/DELETE zenoh operations.

- **`"password"`** (optional) : the user's password.