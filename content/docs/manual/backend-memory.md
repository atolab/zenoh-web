---
title: "Memory backend"
weight : 2210
menu:
  docs:
    parent: manual
---

A backend storing paths/values into an in-memory hashmap.

## Behaviour

The memory backend implementation is not provided by a backend library but is embeded within the [Storages plugin](../plugin-storages) itself.
It is automatically added at zenoh router startup (under `"/@/router/<router-id>/plugin/storages/backend/memory"` admin path)
unless the `--no-backend` argument is used (see [Storages plugin](../plugin-storages#) startup arguments).

Only 1 value per path is stored (the latest one).
If publications arrive un-ordered, the ones that are older than the stored one are ignored.

Note that memory storages can be created at zenoh router startup using the `--mem-storage` argument
(see [Storages plugin](../plugin-storages#) startup arguments).

---------
## Properties for Backend creation

  None.

---------
## Properties for Storage creation

- **`"path_expr"`** (**required**) : the Storage's [Path Expression](../abstractions#path-expression)
