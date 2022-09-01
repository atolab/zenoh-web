---
title: "For a quick test using Docker"
weight : 1015
menu:
  docs:
    parent: getting_started
---

This page describe how to perform a quick test of Zenoh, using a Docker image.

## Run Zenoh router in a Docker container

The Zenoh router is also available in a Docker image. You can deploy a single instance on your local host just running:
```bash
docker run --init -p 7447:7447/tcp -p 8000:8000/tcp eclipse/zenoh
```

The ports used by Zenoh are the following:

  - **7447/tcp** : the Zenoh protocol via TCP
  - **8000/tcp** : the Zenoh REST API

**⚠️ WARNING ⚠️**: _Docker doesn't support UDP multicast between a container and its host (see cases [moby/moby#23659](https://github.com/moby/moby/issues/23659), [moby/libnetwork#2397](https://github.com/moby/libnetwork/issues/2397) or [moby/libnetwork#552](https://github.com/moby/libnetwork/issues/552)). The only case where it works is on Linux using the `--net host` option to make the container to share the host's networking space (i.e. run: `docker run --init --net host eclipse/zenoh`)._  
_The implication of not having UDP multicast working for the Zenoh router is that you need to configure your Zenoh applications (peer or client) with the router's locator as `peer`:_
  - _running the [examples we provide](##pick-your-programming-language), just add the option: `-e tcp/localhost:7447`_
  - _writing your own Zenoh application, you need to add a `connect: {endpoints: ["tcp/localhost:7447"]}}` configuration when initiating the Zenoh API_

### Adding plugins and backends to the container

The Zenoh router supports the dynamic loading of plugins libraries (at startup) and backends libraries (during runtime).  
See the relevant chapters for more details about plugins and backends:
 - [Zenoh plugins](../../manual/plugins)
 - [Zenoh backends and storages](../../manual/backends)

**⚠️ WARNING ⚠️**: _To be compatible with Zenoh in Docker, the libraries must be compiled for **`x86_64-unknown-linux-musl`** target. Look for `.tgz` filenames with this extension when downloading plugins or backends from the [Eclipse zenoh download space](https://download.eclipse.org/zenoh)._

By default the Zenoh router will search for plugins and backends libraries to load in `~/.zenoh/lib`. Thus, to make it able to find the libraries, you can copy them into a `zenoh-docker/lib` directory on your local host and mount the `zenoh-docker` directory as a volume in your container targeting `/root/.zenoh`.

Example:
```bash
docker run --init -p 7447:7447/tcp -p 8000:8000/tcp -v $(pwd)/zenoh-docker:/root/.zenoh eclipse/zenoh
```

Example of a Docker compose file (also configuring Zenoh log level to "debug"):
```bash
version: "3.9"
services:
  zenoh:
    image: eclipse/zenoh
    restart: unless-stopped
    ports:
      - 7447:7447
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

 * Get info of the local Zenoh router:
   ```bash
   curl http://localhost:8000/@/router/local
   ```
 * Add a memory storage on `demo/example/**`:
   ```bash
   curl -X PUT -H 'content-type:application/json' -d '{key_expr:"demo/example/**", volume: "memory"}' http://localhost:8000/@/router/local/config/plugins/storage_manager/storages/demo
   ```
 * Get the storages of the local router (should return the "demo" storage that has just been created):
   ```bash
   curl 'http://localhost:8000/@/router/local/status/plugins/storage_manager/storages/*'
   ```

### Put/Get into Zenoh
Assuming the memory storage has been added, as described above, you can now:

 * Put a key/value into Zenoh:
  ```bash
  curl -X PUT -H 'content-type:text/plain' -d 'Hello World!' http://localhost:8000/demo/example/test
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

Now you can see how to [build your first Zenoh application in Python](../first-app).

## Pick your programming language

If you prefer, you could also have a look to the `examples/zenoh` directory we provide in each Zenoh API:
- [Rust examples](https://github.com/eclipse-zenoh/zenoh/tree/master/zenoh/examples/zenoh)
- [Python examples](https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh)
