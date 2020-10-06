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
docker pull eclipse/zenoh
docker run --init -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp eclipse/zenoh
```

The ports used by zenoh are the following:

  - **7447/tcp** : the zenoh protocol via TCP
  - **7447/udp** : the zenoh scouting protocol using UDP multicast (for clients to automatically discover the router)
  - **8000/tcp** : the zenoh REST API

## First tests using the REST API

The complete Eclipse zenoh's key/value space is accessible through the REST API, using regular HTTP GET, PUT and DELETE methods. In those examples, we use the **curl** command line tool.

### Managing the admin space

 * Get info of the local zenoh router:
   ```bash
   curl http://localhost:8000/@/router/local
   ```
 * Get the backends of the local router (only memory by default):
   ```bash
   curl 'http://localhost:8000/@/router/local/**/backend/*'
   ```
 * Get the storages of the local router (none by default):
   ```bash
   curl 'http://localhost:8000/@/router/local/**/storage/*'
   ```
 * Add a memory storage on `/zenoh/examples/**`:
   ```bash
   curl -X PUT -H 'content-type:application/properties' -d 'path_expr=/zenoh/examples/**' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-storage
   ```

### Put/Get into zenoh
Assuming the memory storage has been added, as described above, you can now:

 * Put a key/value into zenoh:
  ```bash
  curl -X PUT -d 'Hello World!' http://localhost:8000/zenoh/examples/test
  ```
 * Retrieve the key/value:
  ```bash
  curl http://localhost:8000/zenoh/examples/test
  ```
 * Remove the key value
  ```bash
  curl -X DELETE http://localhost:8000/zenoh/examples/test
  ```

## Your first app in Python

Now you can see how to [build your first zenoh application in Python](../first-app).

## Pick your programming language

If you prefer, you could also have a look to the `examples/zenoh` directory we provide in each zenoh API:
- [Rust examples](https://github.com/eclipse-zenoh/zenoh/tree/master/zenoh/examples/zenoh)
- [Python examples](https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh)
