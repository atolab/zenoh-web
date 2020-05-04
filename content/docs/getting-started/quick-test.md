---
title: "For a Quick Test"
weight : 1005
menu:
  docs:
    parent: getting_started
---

This page describe how to perform a quick test of zenoh.

## Run zenoh in Docker

The zenoh infrastructure is also available in a Docker image. You can deploy a single instance on your local host just running:
```bash
docker pull adlinktech/eclipse-zenoh:latest
docker run --init -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp adlinktech/eclipse-zenoh:latest
```

The ports used by zenoh are the following:

  - **7447/tcp** : the zenoh protocol via TCP
  - **7447/udp** : the zenoh scouting protocol using UDP multicast (for clients to automatically discover the router)
  - **8000/tcp** : the zenoh REST API

## First tests using the REST API

The complete Eclipse zenoh's key/value space is accessible through the REST API, using regular HTTP GET, PUT and DELETE methods. In those examples, we use the **curl** command line tool.

### Managing the admin space

 * Get info of the local zenoh router:
   ```
   curl http://localhost:8000/@/router/local
   ```
 * Get the plugins of the local router (http and storages by default):
   ```
   curl 'http://localhost:8000/@/router/local/plugin/*'
   ```
 * Get the backends of the local router (only memory by default):
   ```
   curl 'http://localhost:8000/@/router/local/**/backend/*'
   ```
 * Get the storages of the local router (none by default):
   ```
   curl 'http://localhost:8000/@/router/local/**/storage/*'
   ```
 * Add a memory storage on `/zenoh/examples/**`:
   ```
   curl -X PUT -d '{"selector":"/zenoh/examples/**"}' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-storage
   ```

### Put/Get into zenoh
Assuming the memory storage has been added, as described above, you can now:

 * Put a key/value into zenoh:
  ```
  curl -X PUT -d 'Hello World!' http://localhost:8000/zenoh/examples/test
  ```
 * Retrieve the key/value:
  ```
  curl http://localhost:8000/zenoh/examples/test
  ```
 * Remove the key value
  ```
  curl -X DELETE http://localhost:8000/zenoh/examples/test
  ```

### Using an SQLite3 backend

 * Add the SQLite3 backend using file /tmp/sqlite3-zenoh.db:
   ```
   curl -X PUT -d '{"lib":"sqlite3","url":"sqlite3:///tmp/sqlite3-zenoh.db"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3
   ```
 * Add a storage using the SQLite3 backend on `/zenoh/sql-examples/**`.  
   This storage will store everything in a table named _'example'_, and this table will be truncated when the storage is removed or when zenoh stops:
   ```
   curl -X PUT -d '{"selector":"/zenoh/sql-examples/**","table":"example","on_dispose":"Truncate"}' http://localhost:8000/@/router/local/plugin/storages/backend/sqlite3/storage/my-sql-storage
   ```
 * Put different keys/values into the SQLite3 storage:
   ```
   curl -X PUT -d 'Value A' http://localhost:8000/zenoh/sql-examples/A
   curl -X PUT -d 'Value B' http://localhost:8000/zenoh/sql-examples/B
   curl -X PUT -d 'Value C' http://localhost:8000/zenoh/sql-examples/C
   ```
 * Get all keys/values from the SQLite3 storage:
   ```
   curl 'http://localhost:8000/zenoh/sql-examples/**'
   ```
 * Get all keys/values from the ALL storage:
   ```
   curl 'http://localhost:8000/zenoh/**'
   ```

### Using an InfluxDB backend

 * Add the InfluxDB backend (assuming InfluxDB runs locally, which is by default the case in the zenoh Docker image):
   ```
   curl -X PUT -d '{"lib":"influxdb","url":"http://localhost:8086"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb
   ```
 * Add a storage using the InfluxDB backend on `/zenoh/ifx-examples/**`.  
   This storage will store everything in a databse named _'zenoh'_. If it doesn't exists yet, zenoh creates it. It is configured to drop all series when the storage is removed or when zenoh stops:
   ```
   curl -X PUT -d '{"selector":"/zenoh/ifx-examples/**","db":"zenoh","on_dispose":"DropDB"}' http://localhost:8000/@/router/local/plugin/storages/backend/influxdb/storage/my-influxdb-storage
   ```
 * Put different values for the same key into the InfluxDB storage:
   ```
   curl -X PUT -d '1' http://localhost:8000/zenoh/ifx-examples/A
   curl -X PUT -d '2' http://localhost:8000/zenoh/ifx-examples/A
   curl -X PUT -d '3' http://localhost:8000/zenoh/ifx-examples/A
   ```
 * Get the latest value for this key:
   ```
   curl http://localhost:8000/zenoh/ifx-examples/A
   ```
 * Get the complete serie of values for this key:
   ```
   curl 'http://localhost:8000/zenoh/ifx-examples/A?(stoptime=now())'
   ```
 * Get the values put during the last minute:
   ```
   curl 'http://localhost:8000/zenoh/ifx-examples/A?(starttime=now()-1m)'
   ```
 * Get the values put within the last 30 minutes, but not later than 10 minute ago:
   ```
   curl 'http://localhost:8000/zenoh/ifx-examples/A?(starttime=now()-30m;stoptime=now()-10m)'
   ```

Notice that zenoh supports the complete [InfluxDB time syntax](https://docs.influxdata.com/influxdb/v1.7/query_language/data_exploration/#time-syntax) for the _starttime_ and _stoptime_ properties.


## Your first app in Python

Now you can see how to [build your first zenoh application in Python](../first-app).

## Pick your programming language

If you prefer, you could also have a look to the `examples/zenoh` directory we provide in each zenoh API:
- [Python examples](https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh)
- [Java examples](https://github.com/eclipse-zenoh/zenoh-java/tree/master/examples/zenoh)
- [Go examples](https://github.com/eclipse-zenoh/zenoh-go/tree/master/examples/zenoh)
