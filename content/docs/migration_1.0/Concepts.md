---
title: "Concepts"
weight : 6100
menu:
  docs:
    parent: migration_1.0
---

The Zenoh team have been hard at work preparing an official version 1.0.0 of Zenoh!   
This major release comes with several API changes, quality of life improvements and developer conveniences.

We now have a more stable API and intend to keep backward compatibility in future Zenoh revisions.  
This guide is here to ease the transition to Zenoh 1.0.0 for our users!

## Value is gone, long live ZBytes 
We have replaced `Value` with `ZBytes` and `Encoding`.  
`ZBytes` is the type core to data representation in Zenoh, all API's have be reworked to accept `ZBytes` or something that can be converted into a `ZBytes`.  
We have added a number of conversion implementations for language primitives as well as methods to seamlessly allow user defined structs to be serialized into `ZBytes`.  
`Sample`'s payloads are now `ZBytes`.  `Publisher`, `Queryable` and `Subscriber` now expect `ZBytes` for all their interfaces. The [Attachment](#attachment) API also now accepts a `ZBytes`.

<!-- [key expressions](#key-expression) -->
Each Language bindings will have their own specifics of Serializing and Deserializing, but for the most part it will involve implementing a serialize / deserialize function for your datatype or make use of auto-generated conversions for composite types.

## Encoding

`Encoding` has been reworked, moving away from enumerables to now accepting strings.
While Zenoh does not impose any `Encoding` requirement on the user, providing an `Encoding` can offer automatic wire-level optimization for known types.  
For the user defined `Encoding`, it can be thought of as optional metadata, carried over by Zenoh in such a way that the end user’s application may perform different operations based on `Encoding`.  
We have expanded our list of predefined encoding types from Zenoh 0.11.0 to include variants for numerous IANA standard encodings, including but not limited to  `video/x` , `application/x`, `text/x`, `image/x` and `audio/x` encoding families, as well as an encoding family specific to Zenoh defined by the prefix `zenoh/x` .   
Users can also define their own encoding scheme that does not need to be based on the predefined IANA variants. Upon declaring a specific encoding, the users encoding scheme will be prefixed with `zenoh/bytes` for distinction.


## Attachment

We have made attachment more flexible across API’s, essentially accepting anything that can be converted to a `ZBytes` as an optional extra to `put` , `delete` , on `Query`'s and Query `reply`'s.  
We also allow for composite types to be converted into `ZBytes`, meaning that using the Attachment API as a metadata transport is easier than ever.

## Query & Queryable

The `reply` method of a `Queryable` has gained two variants: `reply_del` and `reply_err` to respectively indicate that a deletion should be performed and that an error occurred.   
Additionally, the 3 variants behave similarly to `put` and `del`, hence providing improved ergonomics.

We have added the ability to get the underlying `Handler` of a Queryable as well.

## Use accessors to get private members
Across language bindings we encapsulate members of structs, and they can’t be accessed directly now.  
The only way to access struct values is to use the getter function associated with them.


## Pull Subscribers have been removed

The concept of a pull subscriber no longer exists in Zenoh.
However, when creating a `Subscriber`, it may be the case that developers only care about the latest data and want to discard the oldest data. 
The `RingChannel` can be used to get a similar behaviour. [Rust Example](https://github.com/eclipse-zenoh/zenoh/blob/main/examples/examples/z_pull.rs)
This contrasts with the `FIFOChannel`, the default channel type used internally in Subscribers, which drops new message once its buffer is full.
You can take a look at examples of usage in any language’s examples/z_pull.x

## Timestamps
Previously we exposed a function to generate timestamps outside of a session.
Due, in part, to our efforts to improve the storage replication logic, users will now have to generate timestamps from a session, with the timestamp inheriting the `ZenohID` of the session.

This will affect user-created plugins and applications that need to generate timestamps in their Storage and sending of data.  
⚠️ Note: Timestamps are important for Storage Alignment (a.k.a. Replication). Data stored in Data bases must include a Timestamp to be properly aligned across Data Stores by Zenoh. 
The `timestamping` configuration option must also be enabled for this.

## Plugins

### Storages
⚠️ Note: The storage-manager will fail to launch if the `timestamping` configuration option is disabled.  
From Zenoh 1.0.0 user-applications can load plugins.  
A, somehow, implicit assumption that dictated the behaviour of storages is that the Zenoh node loading them **has to add a timestamp to any received publication that did not have one**. This functionality is controlled by the `timestamping` configuration option.  
Until Zenoh 1.0.0 this assumption held true as only a router could load storage and the default configuration for a router enables `timestamping`. However, in Zenoh 1.0.0 nodes configured in `client` & `peer` mode can load storage and *their default configuration disables `timestamping`*.

### Plugin Loading

We added the ability for user-applications to load compiled plugins written in Rust, regardless of which language bindings you are using! 

Usage of this feature is [Plugin Loading](#plugin-loading) 

⚠️ Note : When loading a plugin, the Plugin must have been built with the same version of the Rust compiler as the bindings loading it, and the `Cargo.lock` of the plugin must be synced with the same commit of Zenoh.  
This means that if the language bindings are using `rustc` version `1.75`, the plugin must:
- Be built with the same toolchain version `1.75`
- Be built with the same Zenoh Commit
- The plugin `Cargo.lock` have had its packages synced with the Zenoh `Cargo.lock`  

The reason behind this strict set of requirements is due to Rust making no gaurentees regarding data layout in memory.  
This means between compiler versions, the representation may change based on optimizations.  
More on this topic at here : [Rust:Type-Layout](https://doc.rust-lang.org/reference/type-layout.html#representations)

## Config Changes

### Plugin Loading

Loading plugins is achieved by enabling the `plugins_loading` section in config file, with the members `enabled` set to true, and specifying the `search_dirs` for the plugins. 

Directories are specified as object with fields `kind` and `value` is accepted.  
1. If `kind` is `current_exe_parent`, then the parent of the current executable's directory is searched and `value` should be `null`.
    In Bash notation, `{ "kind": "current_exe_parent" }` equals `$(dirname $(which zenohd))` while `"."` equals `$PWD`.
2. If `kind` is `path`, then `value` is interpreted as a filesystem path. Simply supplying a string instead of a object is equivalent to this.  
If `enabled: true` and `search_dirs` is not specified then `search_dirs` falls back to the default value of: 
`".:~/.zenoh/lib:/opt/homebrew/lib:/usr/local/lib:/usr/lib”` 

```jsx
 plugins_loading: {
    // Enable plugins loading.
    enabled: false,
    /// Directories where plugins configured by name should be looked for. Plugins configured by __path__ are not subject to lookup.
    /// If `enabled: true` and `search_dirs` is not specified then `search_dirs` falls back to the default value: ".:~/.zenoh/lib:/opt/homebrew/lib:/usr/local/lib:/usr/lib"
    search_dirs: [{ "kind": "current_exe_parent" }, ".", "~/.zenoh/lib", "/opt/homebrew/lib", "/usr/local/lib", "/usr/lib"],
}
// ... Rest of Config 
```

### Mode Dependent endpoints

Configuration now supports a different list of endpoints depending on the mode zenohd is launched with.
The old behaviour of a single List of endpoints is still supported, applying to `router`, `peer` and `client`, however users can now set endpoints per mode:

```jsx
  connect: {
    /// The list of endpoints to connect to.
    /// Accepts a single list (e.g. endpoints: ["tcp/10.10.10.10:7447", "tcp/11.11.11.11:7447"])
    /// or different lists for router, peer and client 
    endpoints: { router: ["tcp/10.10.10.10:7447"], peer: ["tcp/11.11.11.11:7447"], client: ["tcp/somewhere1::7447", "udp/somewhere2:7447"]  }
},
```

⚠️ Note: in `client` mode, `zenohd` will try connect to each endpoint in order until one is successful, then stop subsequent endpoint connection attempts. `client`'s only connect to a single endpoint. 


### Scouting
 
We have implemented a small change in the configuration syntax concerning the `scouting` section.   
Both `gossip` and `multicast`’s `autoconnect` section's have changed to accept lists of either 
`"peer"`, `"client"` or `"router"`

```jsx
// Zenoh 0.11.0
scouting: {
  multicast: {
    autoconnect: { router: "", peer: "router|peer" },
  },
  gossip: {
    autoconnect: { router: "", peer: "router|peer" },
  },
},

// Zenoh 1.0.0
scouting: {
    multicast: {
      autoconnect: { router: [], peer: ["router", "peer"] },
    },
    gossip: {
      autoconnect: { router: [], peer: ["router", "peer"] },
    },
}
```

Next step is to dive into the migration examples for your favourite language!
