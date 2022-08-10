---
title: "Configuration"
weight : 2050
menu:
  docs:
    parent: manual
---

From version 0.6 of Zenoh, configuration has changed in major ways. This page will take you through the new behaviour of configuration, whether you're using Zenoh as a library, or as an executable through `zenohd`.

# Configuring `zenohd` 
There are 3 ways to configure `zenohd`, which may be used in any combination:
* using a [configuration file](#configuration-files),
* through the [command line arguments](#command-line-arguments),
* and by putting values on the configuration through the [adminspace](#adminspace-configuration).

## Configuration files
`zenohd` has supported configuration files for a long time now, but with version 0.6, we hope to make this the primary interface for configuring your Zenoh infrastructure.

As was the case before, you can specify which configuration file to load with the `--config=/path/to/config/file` CLI argument.
If no path is specified, `zenohd` will use a default configuration instead.

Currently, [JSON5](https://json5.org) and YAML are the primary configuration format (as opposed to v0.5's flat key-value files), but we may add support for other serialization formats in the future.

An example configuration can be read [here](https://github.com/eclipse-zenoh/zenoh/blob/master/EXAMPLE_CONFIG.json5), apart from the `plugins` section, we make an effort to keep the values aligned with the defaults.  
The exact schema for the configuration is the `Config` structure, which can be found in [this file](https://github.com/eclipse-zenoh/zenoh/blob/master/commons/zenoh-config/src/lib.rs).

Don't be alarmed, all of these fields are optional. Only configure the parts that are of interest to you.

We'd like to bring your attention to the `plugins` part of the configuration, as plugin management has also changed a lot with version 0.6.
More on this in the page on [plugins](../plugins).

## Command line arguments
If you want to run `zenohd` with small changes in its configuration, without going through the hassle of writing a new configuration file for it, you may use the `--cfg` CLI argument to edit the configuration.

Specifically, you may use any amount of `--cfg='PATH:VALUE'` arguments to specify the VALUEs you'd like to insert at specific PATHs in the configuration.

PATHs are `/`-separated paths to the part of the configuration you wish to change.
Note for plugins that setting a value in a plugin-less configuration for `plugins/example-plugin/example/path` will result in the recursive creation of the intermediate objects if necessary.

VALUEs must be JSON5-deserializable values: don't forget to surround strings with quotes. Due to this, surrounding the whole `PATH:VALUE` pair with single-quotes is a good practice to avoid parsing errors.

If a value was already present for the specified PATH, it will be replaced with VALUE.

For convenience, some arguments of `zenohd` are provided as shorthands for particularly useful `--cfg` patterns, such as `-P <plugin_name>` which desugars to `--cfg='plugins/<plugin_name>/__required__:true'`.

In case of conflicts, `--cfg` options will override any other sources of configuration for their PATH.

## Reactive configuration
It is possible to register callbacks that will be called when the configuration structure is modified. This lets `zenohd` (or your own application) react to changes in the configuration during runtime.

In the case of `zenohd`, the only user-accessible way of editing the configuration during runtime is through the admin space, as explained a bit [further](#adminspace-configuration) in this page. Whether and how to react to modifications to the configuration file when it exists is still under debate by the core team.

## Adminspace configuration
You can still change elements of a `zenohd` instance's configuration once it's started, by sending put messages to its [admin space](../abstractions#admin-space).

If one of the `zenohd` instances uses the REST plugin to expose Zenoh to HTTP requests, this can be done simply by sending such requests with tools such as `curl`.  
To do this, use commands such as 
```bash
curl -X PUT http://localhost:8000/@/router/local/config/plugins/storage_manager/storages/my-storage -d '{key_expr:"demo/mystore/**", volume:{id:"memory"}}'
#           ^- REST plugin addr ^ ^--- config space --^ ^---- the path to the configured value ---^    ^-------------- the value to insert ----------------^
```

Path-value pairs work much like they do when using [CLI arguments](#command-line-arguments).

Note that while you may attempt to change any part of the configuration through this mean, not all of its properties are actually watched.  
For example, while the storage plugin watches for any change in its configuration and will attempt to act on it, the REST plugin will only log a warning that it observed a change, but won't apply it.  
Changes to non-plugin parts of the configuration may be registered by the configuration, but not acted upon, such as the `mode` field of the configuration which is only ever read at startup.
