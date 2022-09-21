---
title: "Troubleshooting"
weight : 2500
menu:
  docs:
    parent: getting_started
---

## Activate logging

Activating the Zenoh logging can provide useful information for any troubleshooting. The Zenoh router (`zenohd`) and all the Zenoh APIs (except zenoh-pico) are developed with a Rust code base. Logging is controlled via the `RUST_LOG` environment variable that can typically be defined with the desired logging level amongst:
 - `error` - this is the default level if `RUST_LOG` is not defined
 - `warn`
 - `info`
 - `debug`
 - `trace`
 - `off` - to disable all logging

More advanced logging directives can be defined via the `RUST_LOG`. For more details see this [page](https://docs.rs/env_logger/latest/env_logger/#enabling-logging).  
Note that the logs are written on `stderr`.

------
## Known troubles with the Zenoh router (`zenohd`)

### Segmentation fault at startup

The router is likely trying to load an incompatible plugin.

To check this look for such logs:
```C#
[2022-03-28T15:23:36Z INFO  zenohd] Successfully started plugin rest from "/Users/test/.zenoh/lib/libzplugin_rest.dylib"
[2022-03-28T15:23:36Z INFO  zenohd] Successfully started plugin storage_manager from "/Users/test/.zenoh/lib/libzplugin_storage_manager.dylib"
[2022-03-28T15:23:36Z INFO  zenohd] Successfully started plugin webserver from "/Users/test/.zenoh/lib/libzplugin_webserver.dylib"
```
Here you can see all the plugins libraries that have been loaded by `zenohd` at startup. You must check if each of those are using the same Zenoh version as dependency than `zenohd`.  
To assess which one is causing troubles, you can also move or rename all the libraries but one and test if `zenohd` is correctly loading this one. Then repeat the process for each library.


------
## Known troubles with the APIs

### _"Error treating timestamp for received Data"_

If you get such log:
```C#
[2021-08-23T08:44:47Z ERROR zenoh::net::routing::pubsub] Error treating timestamp for received Data (incoming timestamp from E74B6FF3D82D49BEA11B8F1BD0AC4C7A exceeding delta 100ms is rejected: 2021-08-23T08:44:47.299498897Z vs. now: 2021-08-23T08:44:47.069513846Z): drop it!
```

It means your Zenoh application on your local host received a publication from another remote host with a system time that is drifting in the future for more than 100ms with respect to the local system time. You should synchronize the time between your hosts using a tool such as [PTP](http://linuxptp.sourceforge.net/) or [NTP](http://www.ntp.org/).
