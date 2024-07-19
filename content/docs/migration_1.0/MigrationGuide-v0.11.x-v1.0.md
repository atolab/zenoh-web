---
title: "Migrating from Zenoh v0.11.0 to Zenoh v1.0"
weight : 6100
menu:
  docs:
    parent: migration_1.0
---

The engineers at ZettaScale have been hard at work preparing an official version 1.0.0 of Zenoh!   
This major release comes with several API changes, quality of life improvements and developer conveniences.

We now have a more stable API and intend to keep backward compatibility in the future Zenoh revisions.  
This guide is here to ease the transition to Zenoh 1.0.0 for our users!

## Value is gone, long live ZBytes 
We have replaced `Value` with `ZBytes` and `Encoding`.  
`Zbytes` is the type core to data representation in Zenoh, all API's have be reworked to accept `ZBytes` or something that can be converted into a `ZBytes`.  
We have added a number of conversion implementations for language primitives as well as methods to seamlessly allow user defined structs to be serialized into `ZBytes`.  
`Sample`'s payloads are now `ZBytes`, `Publishers`, `Queryables` and `Subscribers` now expect `ZBytes` for all their interfaces.

Each Language bindings will have their own specifics of Serializing and Deserializing, but for the most part it will involve implementing a serialize / deserialize function for your datatype or make use of auto-generated conversations for composite types.

## Encoding

`Encoding` has been reworked. 
Zenoh does not impose any encoding requirement on the user, nor does it operate on it.  
It can be thought of as optional metadata, carried over by Zenoh in such a way that the end user’s application may perform different operations based on encoding.  
We have expanded our list of predefined encoding types from Zenoh 0.11.0 to include variants for numerous IANA standard encodings, including but not limited to  `video/x` , `application/x`, `text/x`, `image/x` and `audio/x` encoding families, as well as an encoding family specific to Zenoh defined by the prefix `zenoh/x` .   
Users can also define their own encoding scheme that does not need to be based on the predefined IANA variants. Upon declaring a specific encoding, the users encoding scheme will be prefixed with `zenoh/bytes` for distinction.


## Attachment

We have made attachment more flexible across API’s, essentially accepting anything that can be converted to a `ZBytes` as an optional extra to `put` , `delete` , on `Query`'s and Query `reply` s.  
We also allow for composite types to be converted into `ZBytes`, meaning that using the Attachment API as metadata is more easy than ever.

## Query & Queryable

`Query` and `Queryable` have been slightly reworked.  
For the API replying to a `Query` from a `Queryable` declared on a session: 
The `reply` function has been split into 3 separate functions variants depending on the type of reply the user wants to send.  
`reply` , `reply_del`, `reply_err`  

We have added the ability to get the underlying `Handler` of a Queryable as well.

## Use accessors to get private members
Across language bindings we encapsulate members of structs, and they can’t be accessed directly now.
The only way to access struct values is to use the getter function associated with them.


## Pull Subscribers have been removed

The concept of a pull subscriber no longer exists in Zenoh,
However, when creating a `Subscriber`, it may be the case that developers only care about the latest data and want to discard the oldest data. 
We can use `RingChannel` to get this behaviour.
This contrasts a `FIFOChannel` which is the default channel type used internally in Subscribers
You can take a look at examples of usage in any language’s examples/z_pull.x

## Timestamps

We now tie generating a timestamp to a Zenoh session, with the timestamp inheriting the `ZenohID` of the session.

This will affect user-created plugins and applications that need to generate timestamps in their Storage and sending of data.  
⚠️ Note: Timestamps are important for Storage Alignment and Replication. Data stored in Data bases must include a Timestamp to be properly aligned across Data Stores by Zenoh. 
The `timestamping` configuration option must also be enabled for this.

## Storages
The storage-manager will fail to launch if the timestamping configuration option is disabled.  
From Zenoh 1.0.0 user-applications can load plugins.  
A, somehow, implicit assumption that dictated the behaviour of storages is that the Zenoh node loading them **has to add a timestamp to any received publication that did not have one**. This functionality is controlled by the `timestamping` configuration option.  
Until Zenoh 1.0.0 this assumption held true as only a router could load storage and the default configuration for a router enables `timestamping`. However, in Zenoh 1.0.0 nodes configured in `client` & `peer` mode can load storage and *their default configuration disables `timestamping`*.

# Config Changes

### Plugin Loading

We added the ability for user-applications to load compiled plugins written in Rust, regardless of  language ! 

Usage of this feature is achieved by simply enabling the `plugins_loading` section in config file with the members `enabled` set to true, and specifying the `search_dirs` for the plugins. 

If no search directories were specified, then the default search directories are 
`".:~/.zenoh/lib:/opt/homebrew/lib:/usr/local/lib:/usr/lib”` 

```jsx
 plugins_loading: {
    // Enable plugins loading.
    enabled: false,
    /// Directories where plugins configured by name should be looked for. Plugins configured by __path__ are not subject to lookup.
    /// If `enabled: true` and `search_dirs` is not specified then `search_dirs` falls back to the default value: ".:~/.zenoh/lib:/opt/homebrew/lib:/usr/local/lib:/usr/lib"
    search_dirs: [],
}
// ... Rest of Config 
```

⚠️ Note : When loading a plugin, the Plugin must have been built with the same version of the Rust compiler as the bindings loading it. 
This means that if the language bindings are using `rustc` version `1.75`, the plugin must be built with the same version.

### Scouting
 
We have implemented a small change in configuration syntax concerning the `scouting` section of config files.   
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