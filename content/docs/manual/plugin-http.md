---
title: "REST plugin"
weight : 3400
menu:
  docs:
    parent: manual
---

The REST plugin provides access to the Zenoh [REST API](../../apis/rest/) by enabling an HTTP server on the Zenoh node where it is running.

------
**Library name:** `zplugin_rest`

------
There are two main ways to start this plugin:
- **Through startup arguments**: `zenohd`'s `--rest-http-port=[PORT | IP:PORT | none]` argument allows you to choose which port will be listened to by the HTTP server. Note that the default value for this argument is `8000`, meaning that unless you specify `none` explicitly, `zenohd` will use this plugin by default.

- **Through configuration**: you may also configure the rest plugin in a `zenohd` config file, as illustrated in the Zenoh repo's [DEFAULT_CONFIG.json5 file](https://github.com/eclipse-zenoh/zenoh/blob/master/DEFAULT_CONFIG.json5)
