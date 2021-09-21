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
 - on MacOS: `libzbackend_<name>.dylib`
 - on Windows: `zbackend_<name>.dll`

The list of paths in which the Storages plugin will search for backends can be configured via the `--backend-search-dir` option (that can be repeated to specify several directories). The default list can be seen in using the `--help` option of the zenoh router.

Note that by default the Storages plugin already embeds and provides at startup the [Memory backend](../backends-list#memory-backend).

------
**Library name:** `zplugin_storages`

------
**Startup arguments** (added to zenoh router's startup arguments):
 - `--backend-search-dir=[DIRECTORY]` : A directory where to search for backends libraries to load.
                             Repeat this option to specify several search directories.
 - `--no-backend`: If true, no backend (and thus no storage) are created at startup.
                   If false (default) the Memory backend it present at startup.
 - `--mem-storage=[PATH_EXPR]` : A memory storage to be created at start-up. Repeat this option to created several storages.

