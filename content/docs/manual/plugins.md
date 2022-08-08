---
title: "Zenoh plugins"
weight : 2100
menu:
  docs:
    parent: manual
---

The zenoh router (`zenohd` executable) supports the loading of plugins at start-up.

A zenoh plugin is a library that can be loaded by the zenoh router at start-up. It shares a runtime with it, allowing the plugin to use the regular zenoh rust APIs with the same peer ID.

Zenoh already provides the following plugins in its default repository:
 - the [REST plugin](../plugin-http): providing the zenoh REST API
 - the [Storage Manager plugin](../plugin-storage-manager): providing management of [storages](../abstractions#storage)

## Before v0.6
By default the zenoh router will automatically search for and load plugins library files with these names:
 - on Unix/Linux: `libzplugin_*.so`
 - on macOS: `libzplugin_*.dylib`
 - on Windows: `zplugin_*.dll`

The list of paths in which the zenoh router will search for plugins can be configured via the `--plugin-search-dir` option (this can be repeated to specify several directories). The default list can be seen using the `--help` option.

This automatic search and load of plugins can be deactivated using the `--plugin-nolookup` option.  
Plugin library files to load manually can be specified using the `--plugin` option (repeatable). In this case, the complete path of the library file must be specified, and its filename can be anything.

## From v0.6 
Zenoh 0.6 has had its configuration and plugin infrastructure overhauled. The most major change is that **`zenohd` no longer loads all available plugins at startup**.

Instead, only plugins that appear in the configuration are loaded. If a plugin is added to the configuration during runtime (for example through the [admin space](../abstractions#admin-space)), it will be loaded then.

This choice was made to reduce side effects, as loading all available plugins can lead to loading plugins that have behaviour you do not expect, or that may have weird interractions when running side-by-side.

The configuration has 2 fields that pertain to plugins:
* `plugin_search_dirs`, where you may specify a list of directories where `zenohd` should look for plugins that have been requested by name.
* `plugins`, where you may specify which plugins you require, as well as provide configuration for them.

Plugins can no longer add CLI arguments to those of `zenohd`. Instead, they are expected to obtain the information they need to run through the new configuration infrastructure.  
The `--plugin...` arguments have also seen their purpose slightly changed:
* `--plugin-nolookup` no longer exists, as this is now the normal behaviour of `zenohd`.
* `--plugin-search-dir` now replaces the search directories specified through configuration.
* `--plugin [VALUE]` now inserts a plugin into the configuration. If VALUE is a path, it will be requested by path. Otherwise, it will be requested by name. When a plugin is requested by `<name>`, `zenohd` will look for the system-appropriate `zplugin_<name>` dynamic library file within the `plugin_search_dirs`.

### The `plugins` configuration field
This field may contain a dictionary, where each key is the configured plugin's name, and the associated value is a dictionary holding its configuration.

In this latter dictionary, 2 properties are reserved by `zenohd`:
* `__path__` may hold a string or list of strings, which are the paths where the plugin is expected to be located. If this option is defined, lookup-by-name is disabled for the requested plugin, and the first path to resolve in a successful load will be used as the plugin's path.
* `__required__` may hold a boolean value. If set to `true`, `zenohd` will panic if unable to load the requested plugin. Otherwise, it will simply log an error. Plugins are encouraged to look to this field to assertain whether they should be allowed to panic or not.

The rest of the dictionary may be used by plugins as they see fit. If elements of their configuration are modified at runtime, plugins will be given a chance to observe the modification and interfere with it.