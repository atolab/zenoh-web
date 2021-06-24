---
title: "Zenoh plugins"
weight : 2100
menu:
  docs:
    parent: manual
---

The zenoh router (`zenohd` executable) supports the loading of plugins at start-up.

A zenoh plugin is a library that can be loaded by the zenoh router at start-up. It shares a runtime with it, allowing the plugin to use the regular zenoh and/or zenoh-ne APIs with the same peer ID.

By default, the zenoh router automatically searches and loads plugin library files with the following names:
 - on Unix/Linux: `libzplugin_*.so`
 - on MacOS: `libzplugin_*.dylib`
 - on Windows: `zplugin_*.dll`

The list of paths the zenoh router uses to search for plugins can be configured via the `--plugin-search-dir` option (that can be repeated to specify several directories). To see the default list, use the `--help` option.

To deactivate the automatic search and load of plugins, use the `--plugin-nolookup` option.  
To specify which plugins library files to load, use the `--plugin` option (repeatable). The complete path of the library file must be specified, and its filename is free.

Zenoh already provides the following plugins:
 - the [REST plugin](../plugin-rest): providing the zenoh REST API
 - the [Storages plugin](../plugin-storages): providing management of [storages](../abstractions#storage) and [backends](../abstractions#backend)
