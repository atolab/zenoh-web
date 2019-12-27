---
title: "API documentations"
weight : 9000
menu: "docs"
---

All the client APIs documentations are avaliable on [Read the Docs](https://readthedocs.org/search/?q=zenoh):

## Python
https://readthedocs.org/projects/zenoh-python/

## Java
https://readthedocs.org/projects/zenoh-java/

## Go
https://readthedocs.org/projects/zenoh-go/

--------------------

## REST API

Zenoh also offers a REST API via the zenoh-http plugin. When starting zenoh with default options,
this HTTP plugin is automatically started on port 8000 and ready to answer HTTP requests.  
The full zenoh key/value space is accessible via this REST API, including the [Admin space](../../manual/abstractions#admin-space).

### GET

Binds to the **get(selector)** operation on zenoh.

- **URL**:     `http://host:8000/selector`
- **body**:    none
- **headers**: none

The results are returned as a JSON array of objects containing `"key"`, `"value"`and `"time"`.

Examples using curl:

```bash
# Get the keys/values matching /demo/**
curl http://localhost:8000/demo/**
[
{ "key": "/demo/example/zenoh-python-put",
  "value": "Put from Zenoh Python!",
  "time": "2019-11-26T10:04:06.369743108Z" },
{ "key": "/demo/example/zenoh-java-put",
  "value": "Put from Zenoh Java!",
  "time": "2019-11-26T10:04:15.031682968Z" },
{ "key": "/demo/example/zenoh-go-put",
  "value": "Put from Zenoh Go!",
  "time": "2019-11-26T10:04:09.879540920Z" }
]

# Get the keys/values matching /demo/example/*eval (i.e. the zenoh eval examples)
# with property name=Bob
curl http://localhost:8000/demo/example/*eval?(name=Bob)
[
{ "key": "/demo/example/zenoh-java-eval",
  "value": "Eval from Bob",
  "time": "None" }
]

# Get the list of storages via a Get on admin space (/@/...)
curl -g http://localhost:8000/@/**/storage/**
[
{ "key": "/@/router/cd5723ef1abe4a8f9984d003283c7e28/plugin/storages/backend/sqlite3/storage/sto-65069",
  "value": "on_dispose=Truncate;selector=/demo/example/sql/**;table=example",
  "time": "2019-11-26T11:22:39.668893098Z" },
{ "key": "/@/router/cd5723ef1abe4a8f9984d003283c7e28/plugin/storages/backend/influxdb/storage/sto-80720",
  "value": "db=example;on_dispose=DropDB;selector=/demo/example/influx/**",
  "time": "2019-11-26T11:22:39.572923898Z" },
{ "key": "/@/router/cd5723ef1abe4a8f9984d003283c7e28/plugin/storages/backend/memory/storage/Demo",
  "value": "selector=/demo/example/**",
  "time": "2019-11-26T11:22:08.759634017Z" }
]
```



### PUT

Binds to the **put(path,value)** operation on zenoh.

- **URL**:     `http://host:8000/path`
- **body**:    the value to put
- **headers**: 
   - **content-type**: the value encoding (optional; unspecified implies RAW encoding)

The supported content-types and their mapping to zenoh encoding are:

| content-type                      | zenoh encoding |
| --------------------------------- | -------------- |
| text                              | string         |
| application/x-www-form-urlencoded | string         |
| application/xml                   | string         |
| application/xhtml+xml             | string         |
| application/json                  | JSON           |
| *any other value*                 | RAW            |
| *unspecified*                     | RAW            |

Examples using curl:

  ```bash
  # Put a string value in /demo/example/test
  curl -X PUT -d 'Hello World!' http://localhost:8000/demo/example/test

  # Put a JSON value in /demo/example/json
  curl -X PUT -H "Content-Type: application/json" -d '{"value": "Hello World!"}' http://localhost:8000/demo/example/test

  # Create a Memory storage on /demo/test/** via a Put on admin space (/@/...)
  curl -X PUT -d '{"selector":"/demo/test/**"}' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-test
  ```

### DELETE

Binds to the **remove(path)** operation on zenoh.

- **URL**:     `http://host:8000/path`
- **body**:    none
- **headers**: none

Examples using curl:

  ```bash
  # Remove the value with key /demo/example/test
  curl -X DELETE http://localhost:8000/demo/example/test

  # Remove a storage via a Remove on admin space (/@/...)
  curl -X DELETE http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-test
  ```

