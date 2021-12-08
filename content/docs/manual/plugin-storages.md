---
title: "Storages plugin"
weight : 2120
menu:
  docs:
    parent: manual
---

The Storages plugin provides the management of [Backends](../abstractions#backend) and [Storages](../abstractions#storage).  
It allows you at runtime to dynamically add/remove backends and storages. More details in [Zenoh backends](../backends) chapter.

When adding a backend, a short name of the backend library can be provided instead of the complete path to the library file.
In such case the Storages plugin will search and load the library file with such name:
 - on Unix/Linux: `libzbackend_<name>.so`
 - on macOS: `libzbackend_<name>.dylib`
 - on Windows: `zbackend_<name>.dll`

The list of paths in which the Storages plugin will search for backends can be configured via the `--backend-search-dir` option (that can be repeated to specify several directories). The default list can be seen in using the `--help` option of the zenoh router.

Note that by default the Storages plugin already embeds the [Memory backend](../backends-list#memory-backend).

------
**Library name:** `zplugin_storages`

------
## Until v0.5
**Startup arguments** (added to zenoh router's startup arguments):
- `--backend-search-dir=[DIRECTORY]` : A directory where to search for backends libraries to load.
                            Repeat this option to specify several search directories.
- `--no-backend`: If true, no backend (and thus no storage) are created at startup.
                  If false (default) the Memory backend is present at startup.
- `--mem-storage=[PATH_EXPR]` : A memory storage to be created at start-up. Repeat this option to create several storages.

## From v0.6
Storages now use structured [configurations](../configuration) to require the creation and deletion of backends and storages.  
The main schema is as follows:
```typescript
{
  // Search directories when backends are requested by name
  backends_search_dirs?: string | string[], 
  // The list of backends that must be created
  backends: {
    // All backends must have unique names
    "<backend_name>": {
      // Much like for plugins, if a list of paths is configured, the backend will be run using the first path pointing to a loadable library.
      // If no path is specified, a matching library for <backend_name> will be searched in the configured directories.
      __path__?: string | string[],
      storages: {
        "<storage_name>": {
          // The key expression that the storage should be set up for, such as "/demo/storage/**"
          key_expr: string,
          // A prefix of `key_expr` that should be stripped from it when storing keys in the storage
          strip_prefix?: string,
          // some storages may hold options interpreted only by specific backends, such as `base_dir` for filesystem backends
          ...storage_specifics: any,
        }
      },
      // backends may have options specific to them, such as `url` and `db` for the influxdb backends
      ...backend_specifics: any, 
    }
  }
}
```

Note that backends will automatically be created if necessary when you configure a new storage. You may however configure a backend without its children storages first in order to set up its eventual specifics.

The `memory` backend is always available, as it is statically linked into the Storages plugin. Its name is always `memory`, and it has no backend or storage specific configuration options of its own.