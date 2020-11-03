---
title: "API documentations"
weight : 9000
menu: "docs"
---

All the client APIs documentations are avaliable on [Read the Docs](https://readthedocs.org/search/?q=zenoh):

## Rust
https://zenoh-rust.readthedocs.io/

## C
https://zenoh-c.readthedocs.io/

## Python
https://zenoh-python.readthedocs.io/


--------------------

## REST API

Zenoh also offers a REST API via the zenoh-http plugin. When starting zenoh with default options,
this HTTP plugin is automatically started on port 8000 and ready to answer HTTP requests.  
The full zenoh key/value space is accessible via this REST API, including the Admin Space under the `'/@'`prefix.

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
  "time": "2019-11-26T10:04:06.369743108Z/BEAFA6F367E24D27B2505DF2A971B21C" },
{ "key": "/demo/example/zenoh-java-put",
  "value": "Put from Zenoh Java!",
  "time": "2019-11-26T10:04:15.031682968Z/BEAFA6F367E24D27B2505DF2A971B21C" },
{ "key": "/demo/example/zenoh-go-put",
  "value": "Put from Zenoh Go!",
  "time": "2019-11-26T10:04:09.879540920Z/BEAFA6F367E24D27B2505DF2A971B21C" }
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
{ "key": "/@/router/DA087EDE87EF442A953C31F64A195D7C/plugin/storages/backend/memory/storage/mem-storage-1",
  "value": "{"path_expr":"/demo/demo/**"}",
  "time": "None" }
]
```



### PUT

Binds to the **put(path,value)** operation on zenoh.

- **URL**:     `http://host:8000/path`
- **body**:    the value to put
- **headers**: 
   - **content-type**: the value encoding (optional; unspecified implies "application/octet-stream" encoding. Note that curl by default uses "application/x-www-form-urlencoded")

The values with the following content-types will be automatically converted by zenoh into a typed Value:

| content-type             | zenoh Value type |
| ------------------------ | ---------------- |
| text/plain               | StringUTF8       |
| application/properties   | Properties       |
| application/json         | Json             |
| text/json                | Json             |
| application/integer      | Integer          |
| application/float        | Float            |

Examples using curl:

  ```bash
  # Put a string value in /demo/example/test
  curl -X PUT -d 'Hello World!' http://localhost:8000/demo/example/test

  # Put a JSON value in /demo/example/json
  curl -X PUT -H "Content-Type: application/json" -d '{"value": "Hello World!"}' http://localhost:8000/demo/example/test

  # Create a Memory storage on /demo/test/** via a Put on admin space (/@/...)
  curl -X PUT -H 'content-type:application/properties' -d 'path_expr=/demo/test/**' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-test
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

