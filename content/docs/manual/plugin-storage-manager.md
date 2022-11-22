---
title: "Storage manager plugin"
weight : 3500
menu:
  docs:
    parent: manual
---

The `storage_manager` plugin provides `zenohd` with the ability to store values associated with a set of keys, allowing other nodes to query the most recent values associated with these keys.

------
**Library name:** `zplugin_storage_manager`

------


## Backends and Volumes
Since there exist many ways for a Zenoh node to store values it may need to serve later, the storage manager plugin relies on dynamically loaded "backends" to provide this functionality. Typically, a backend will leverage some third-party technology, such as databases, to handle storage. A possibly convenient side effect of using databases as backends is that they may also be used as an interface between your Zenoh infrastructure and an external infrastructure that may interact independently with the database.

You may want to load the same backend multiple times with different configurations: we refer to these instances as "volumes". These volumes can in turn be relied on by any number of storages, just like you can store many files on a filesystem volume.

When defining volumes, there are multiple ways to inform it of which backend it should use:
- With the `backend` option, you may specify the name of a backend that will be used for lookup.
- With the `__path__` option, you may specify a list of absolute paths. The storage manager will then load the first file to be found at one of these paths. Specifying this option disables name-based lookup completely.
- Using neither of these options will result in the same name-based lookup as with the `backend` option, using the volume's name.

This name-based lookup consists in searching the configured `backends_search_dirs` for a `zbackend_<name>` dynamic library file; the exact searched filenames are platform-specific:
 - on Unix/Linux: `libzbackend_<name>.so`
 - on macOS: `libzbackend_<name>.dylib`
 - on Windows: `zbackend_<name>.dll`

### Integrated volumes
By design, the storage manager may make some volumes available regardless of configuration. Currently, the only such volume is the `memory` volume. This volume stores key-value pairs in RAM. While convenient, keep in mind that this storage is not persistent through restarts of `zenohd`.

## Configuration
Storages use structured [configurations](../configuration) to require the creation and deletion of backends and storages.  
The main schema is as follows:
```typescript
{
  // Search directories when backends are requested by name
  backends_search_dirs?: string | string[], 
  // The list of volumes that must be created
  volumes: {
    // All volumes on a Zenoh node must have unique names
    "<volume_name>": {
      // The name of the backend this volume should run on. If unspecified, it defaults to the name of the volume.
      backend?: string,
      // Much like for plugins, if a list of paths is configured, the backend will be run using the first path pointing to a loadable library.
      // If unspecified, name-based lookup will be used instead.
      __path__?: string | string[],
      // backends may have options specific to them, such as `url` for the influxdb backends
      ...backend_specifics: any, 
    }
  },
  storages: {
    // All storages must also have unique names.
    "<storage_name>": {
      // The key expression that the storage should be set up for, such as "demo/storage/**"
      key_expr: string,
      // A prefix of `key_expr` that should be stripped from it when storing keys in the storage
      strip_prefix?: string,
      // Storages depend on a volume to do the actual storing of data.
      // `volume: "memory"` is equivalent to `volume: {id: "memory"}`, but some volumes may require additional configuration. For example, a volume running on the `filesystem` backend needs each storage to specify a `base_dir`.
      volume: string | { id: string, ...configuration: any}
    }
  }
}
```
The example configuration provided [here](https://github.com/eclipse-zenoh/zenoh/blob/master/DEFAULT_CONFIG.json5) has concrete examples of backend and storage configuration.

## Backends list
Here is a list of the available backends:

| Backend     | Description                                               | GitHub repo & doc                         |
|-------------|-----------------------------------------------------------|-------------------------------------------------|
| S3          | Storages in S3 databases.                                 | [eclipse-zenoh/zenoh-backend-s3]          |
| InfluxDB    | Storages in InfluxDB databases.                           | [eclipse-zenoh/zenoh-backend-influxdb]    |
| RocksDB     | Storages in RocksDB databases.                            | [eclipse-zenoh/zenoh-backend-rocksdb]     |
| File System | Storages on local files system, each key/value in a file. | [eclipse-zenoh/zenoh-backend-filesystem]  |

[eclipse-zenoh/zenoh-backend-s3]: https://github.com/eclipse-zenoh/zenoh-backend-s3
[eclipse-zenoh/zenoh-backend-influxdb]: https://github.com/eclipse-zenoh/zenoh-backend-influxdb
[eclipse-zenoh/zenoh-backend-rocksdb]: https://github.com/eclipse-zenoh/zenoh-backend-rocksdb
[eclipse-zenoh/zenoh-backend-filesystem]: https://github.com/eclipse-zenoh/zenoh-backend-filesystem

-------------------------------

## Volumes and Storage management during runtime
The Volumes and Storages of a Zenoh router can be managed at runtime via its admin space, if it's configured to be writeable:
 - either via the configuration file in the `adminspace.permissions` section
 - either via the `zenohd` command line option: `--adminspace-permissions <[r|w|rw|none]>`

The most convenient way to edit configuration at runtime is through the [admin space](../configuration#adminspace-configuration), via the [REST API](../../apis/rest). This is the method we will teach here through `curl` commands. If you're unfamiliar with `curl`, it's a command line tool to make HTTP requests, here's a quick catch-up, with the equivalent in JS's standard library's `fetch`:
```bash
curl -X PUT        'http://hostname:8000/key/expression' -H 'content-type:application/json'      -d '{"some": ["json", "data"]}'
#    ^HTTP METHOD  ^Target URL for the request           ^Header saying the data is in JSON      ^The body of the request
```
```javascript
fetch('http://hostname:8000/key/expression', {
      method: 'PUT',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({some: ["json", "data"]})
    })
```

### Adding a volume
To create a volume, add an entry describing it to the configuration. Let's say you'd like to create "my-volume" on a node that had the influxdb daemon listen on port 8086:

```bash
curl -X PUT 'http://hostname:8000/@/router/local/config/plugins/storage_manager/volumes/my-volume' -H 'content-type:application/json' -d '{"backend": "influxdb", "url": "http://localhost:8086"}'
```

Note that if you only planned on having a single volume relying on influxdb, you might as well name that volume "influxdb", saving you the need to specify that the "influxdb" backend is the one you want to use:

```bash
curl -X PUT 'http://hostname:8000/@/router/local/config/plugins/storage_manager/volumes/influxdb' -H 'content-type:application/json' -d '{"url": "http://localhost:8086"}'
```

### Removing a volume
To remove a volume, simply delete its entry from the configuration with:

```bash
curl -X DELETE 'http://hostname:8000/@/router/local/config/plugins/storage_manager/volumes/my-volume'
```

The storage manager will delete the associated storages as well.


### Adding a storage
You can add storages at any time by adding them to the configuration through the admin space.

The simplest volume to use is the integrated "memory" volume, since it requires no extra configuration. Let's have "my-storage" work on `demo/my-storage/**`:
```bash
curl -X PUT 'http://hostname:8000/@/router/local/config/plugins/storage_manager/storages/my-storage' -H 'content-type:application/json' -d '{"key_expr": "demo/my-storage/**", "volume": "memory"}'
```

Some volumes, like that "my-volume" one we created [earlier](#adding-a-volume), need a bit more configuration. Any volume supported by the `influxdb` backend, for example, needs to know on what database to store the data associated with each storage through a `db` argument:
```bash
curl -X PUT 'http://hostname:8000/@/router/local/config/plugins/storage_manager/storages/my-other-storage' -H 'content-type:application/json' -d '{"key_expr": "demo/my-other-storage/**", "volume": {"id": "my-volume", "db": "MyOtherStorage"}}'
```

### Removing a storage
Just like volumes, removing a storage is as simple as deleting its entry in the configuration. Note that removing a volume's last storage will not remove that volume: volumes with 0 storages depending on them are perfectly legal.

```bash
curl -X DELETE 'http://hostname:8000/@/router/local/config/plugins/storage_manager/storages/my-storage'
```

### Checking a volume's or a storage's status
**TODO**: this part hasn't been redocumented yet. Feel free to contact us on [Discord](https://discord.gg/cY4nVjUd) to get more information.
