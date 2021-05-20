---
title: "Storages plugin"
weight : 2120
menu:
  docs:
    parent: manual
---

The Storages plugin provides the management of [Backends](../abstractions#backend) and [Storages](../abstractions#storage).  
At runtime it allows you to dynamically add/remove backends and storages. For more information, refer to [Zenoh backends](../backends).

When you add a backend, a short name of the backend library can be provided instead of the complete path to the library file.
In this case the Storages plugin searches and loads the library file with the following name:

 - on Unix/Linux: `libzbackend_<name>.so`
 - on MacOS: `libzbackend_<name>.dylib`
 - on Windows: `zbackend_<name>.dll`

The list of paths in which the Storages plugin searches for backends can be configured via the `--backend-search-dir` option (that can be repeated to specify several directories). To see the default list, use the `--help` option on the zenoh router.

Note that by default the Storages plugin already embeds and provides the [Memory backend](../backend-memory) at startup.

------
**Library name:** `zplugin_storages`

------
**Startup arguments** (added to zenoh router's startup arguments):
 - `--backend-search-dir=[DIRECTORY]`: The directory to search for backends libraries to load. Repeat this option to specify several search directories.
 - `--no-backend`: If true, no backends (and thus no storage) are created at startup. If false (default) the Memory backend it present at startup.
 - `--mem-storage=[PATH_EXPR]`: A memory storage to create at start-up. Repeat this option to create several storages.

