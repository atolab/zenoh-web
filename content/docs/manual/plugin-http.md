---
title: "REST plugin"
weight : 2110
menu:
  docs:
    parent: manual
---

The REST plugin provides access to the zenoh [REST API](../../apis/apis/#rest-api).

------
**Library name:** `zplugin_rest`

------
**Startup arguments** (added to zenoh router's startup arguments):
 - `--rest-http-port=[PORT]` : The REST plugin's http port (default: 8000)
