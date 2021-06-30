---
title: "For a Quick Test using Docker"
weight : 1015
menu:
  docs:
    parent: getting_started
---

This page describe how to perform a quick test of zenoh, using a Docker image.

## Run zenoh router in a Docker container

The zenoh router is also available in a Docker image. You can deploy a single instance on your local host just running:
```bash
docker run --init -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp eclipse/zenoh
```

The ports used by zenoh are the following:

  - **7447/tcp** : the zenoh protocol via TCP
  - **7447/udp** : the zenoh scouting protocol using UDP multicast (for clients to automatically discover the router)
  - **8000/tcp** : the zenoh REST API

### Adding plugins and backends to the container

The zenoh router supports the dynamic loading of plugins libraries (at startup) and backends libraries (during runtime).  
See the relevant chapters for more details about plugins and backends:
 - [Zenoh plugins](../../manual/plugins)
 - [Zenoh backends and storages](../../manual/backends)

**WARNING:** To be compatible with zenoh in Docker, the libraries must be compiled for **`x86_64-unknown-linux-musl`** target.
Look for `.tgz` filenames with this extension when downloading plugins or backends from the
[Eclipse zenoh download space](https://download.eclipse.org/zenoh).

By default the zenoh router will search for plugins and backends libraries to load in `~/.zenoh/lib`. Thus, to make it able to find the libraries, you can copy them into a `zenoh-docker/lib` directory on your local host and mount the `zenoh-docker` directory as a volume in your container targeting `/root/.zenoh`.

Example:
```bash
docker run --init -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp -v $(pwd)/zenoh-docker:/root/.zenoh eclipse/zenoh
```

Example of a Docker compose file (also configuring zenoh log level to "debug"):
```bash
version: "3.9"
services:
  zenoh:
    image: eclipse/zenoh
    restart: unless-stopped
    ports:
      - 7447:7447
      - 7447:7447/udp
      - 8000:8000
    volumes:
      - ./zenoh_docker:/root/.zenoh
    environment:
      - RUST_LOG=debug
```


--------------------------------
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
 * Add a memory storage on `/demo/example/**`:
   ```bash
   curl -X PUT -H 'content-type:application/properties' -d 'path_expr=/demo/example/**' http://localhost:8000/@/router/local/plugin/storages/backend/memory/storage/my-storage
   ```

### Put/Get into zenoh
Assuming the memory storage has been added, as described above, you can now:

 * Put a key/value into zenoh:
  ```bash
  curl -X PUT -d 'Hello World!' http://localhost:8000/demo/example/test
  ```
 * Retrieve the key/value:
  ```bash
  curl http://localhost:8000/demo/example/test
  ```
 * Remove the key value
  ```bash
  curl -X DELETE http://localhost:8000/demo/example/test
  ```

## Your first app in Python

Now you can see how to [build your first zenoh application in Python](../first-app).

## Pick your programming language

If you prefer, you could also have a look to the `examples/zenoh` directory we provide in each zenoh API:
- [Rust examples](https://github.com/eclipse-zenoh/zenoh/tree/master/zenoh/examples/zenoh)
- [Python examples](https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh)
