---
title: "Zenoh plugins"
weight : 2100
menu:
  docs:
    parent: manual
---

The zenoh router (`zenohd` executable) supports the loading of plugins at start-up.

A zenoh plugin is a library that can be loaded by the zenoh router at start-up. It shares a runtime with it, allowing the plugin to use the regular zenoh and/or zenoh-net APIs with the same peer ID.

By default the zenoh router will automatically search for and load plugins library files with these names:
 - on Unix/Linux: `libzplugin_*.so`
 - on MacOS: `libzplugin_*.dylib`
 - on Windows: `zplugin_*.dll`

The list of paths in which the zenoh router will search for plugins can be configured via the `--plugin-search-dir` option (this can be repeated to specify several directories). The default list can be seen using the `--help` option.

This automatic search and load of plugins can be deactivated using the `--plugin-nolookup` option.  
Plugin library files to load manually can be specified using the `--plugin` option (repeatable). In this case, the complete path of the library file must be specified, and its filename can be anything.

Zenoh already provides the following plugins:
 - the [REST plugin](../plugin-http): providing the zenoh REST API
 - the [Storages plugin](../plugin-storages): providing management of [storages](../abstractions#storage) and [backends](../abstractions#backend)
