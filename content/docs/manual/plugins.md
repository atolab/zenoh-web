---
title: "Zenoh plugins"
weight : 2100
menu:
  docs:
    parent: manual
---

The zenoh router (`zenohd` executable) supports the loading of plugins at start-up.

A zenoh plugin is a library that can be loaded by the zenoh router at start-up. It shares a runtime with it, allowing the plugin to use the regular zenoh and/or zenoh-ne APIs with the same peer ID.

By default the zenoh router will automatically search and load plugins library files with such names:
 - on Unix/Linux: `libzplugin_*.so`
 - on MacOS: `libzplugin_*.dylib`
 - on Windows: `zplugin_*.dll`

The list of paths in which the zenoh router will search for plugins can be configured via the `--plugin-search-dir` option (that can be repeated to specify several directories). The default list can be seen in using the `--help` option.

This automatic search and load of plugins can be desactivated using the `--plugin-nolookup` option.  
And some plugins library files to load anyway can be specified using the `--plugin` option (repeatable). In such case, the complete path of the library file must be specified, and its filename is free.

Zenoh already provides the following plugins:
 - the [REST plugin](../plugin-http): providing the zenoh REST API
 - the [Storages plugin](../plugin-storages): providing management of [storages](../abstractions#storage) and [backends](../abstractions#backend)
