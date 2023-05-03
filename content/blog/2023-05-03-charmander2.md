---
title: "Zenoh Charmander Grows Stronger"
date: 2023-05-03
menu: "blog"
weight: 20230503
description: "May 3rd, 2023 -- Paris"
draft: false
---

The new [Zenoh](https://zenoh.io) Charmander 0.7.1-rc is out and comes with some aces up the sleeve! This patch release marks as stable the following APIs:

- C++ API
- Query payload

It introduces some new features:

- C++ API is now compatible with both zenoh-c and zenoh-pico
- TLS supports now Let’s Encrypt and IP-based certificates
- Key formatters
- Transport protocol whitelisting

It brings to life these experimental features:

- Zenoh and ROS1 bridge
- Liveliness assertion

And it also comes with a load of various bug fixes and improvements.

## C++ API

[Zenoh Charmander 0.7.0-rc](https://zenoh.io/blog/2023-01-10-zenoh-charmander/#c-bindings) introduced experimental support for [C++ bindings](https://zenoh.io/blog/2023-01-10-zenoh-charmander/#c-bindings), which are built on top of the Zenoh C API. This release introduces the support of [zenoh-pico](https://github.com/eclipse-zenoh/zenoh-pico) in addition to the already supported [zenoh-c](https://github.com/eclipse-zenoh/zenoh-c). This means you can now write your Zenoh C++ application and run it on any embedded platform supported by [zenoh-pico](https://github.com/eclipse-zenoh/zenoh-pico#zenoh-pico-native-c-library-for-constrained-devices)! You can check [here](https://github.com/eclipse-zenoh/zenoh-cpp#library-usage) how to set up zenoh-cpp as a library and switch between zenoh-c and zenoh-pico.

Moreover, this release finally provides the full coverage of the Zenoh stable API and the build for zenoh C/C++ projects was made simpler and more flexible. Now zenoh-c, zenoh-pico and zenoh-cpp can be directly included into the parent CMake project without installation - see examples [here](https://github.com/eclipse-zenoh/zenoh-cpp/tree/main/examples/simple). The full set of C++ examples is available [here](https://github.com/eclipse-zenoh/zenoh-cpp/tree/main/examples/).

## Query payload

The [query payload API](https://zenoh.io/blog/2023-01-10-zenoh-charmander/#query-payload) has been marked stable and added to all Zenoh examples: in Rust ([get](https://github.com/eclipse-zenoh/zenoh/blob/eca888b410a0afb1df939393d4a304b7f926cd5e/examples/examples/z_get.rs#L32) and [queryable](https://github.com/eclipse-zenoh/zenoh/blob/eca888b410a0afb1df939393d4a304b7f926cd5e/examples/examples/z_queryable.rs#L49)), C ([get](https://github.com/eclipse-zenoh/zenoh-c/blob/2779f7ac470b1557e6fd8fd0ebfd04002ee8db2c/examples/z_get.c#L60) and [queryable](https://github.com/eclipse-zenoh/zenoh-c/blob/2779f7ac470b1557e6fd8fd0ebfd04002ee8db2c/examples/z_queryable.c#L32)), C++ ([get](https://github.com/eclipse-zenoh/zenoh-cpp/blob/81e1a69693bb45d4f23236d769ff4f4ad74ddddb/examples/universal/z_get.cxx#L62) and [queryable](https://github.com/eclipse-zenoh/zenoh-cpp/blob/81e1a69693bb45d4f23236d769ff4f4ad74ddddb/examples/universal/z_queryable.cxx#L75)), Python ([get](https://github.com/eclipse-zenoh/zenoh-python/blob/78a3902316f0e99d593ff7a2013ef920e637fa60/examples/z_get.py#L82) and [queryable](https://github.com/eclipse-zenoh/zenoh-python/blob/78a3902316f0e99d593ff7a2013ef920e637fa60/examples/z_queryable.py#L73)), and Zenoh-Pico ([get](https://github.com/eclipse-zenoh/zenoh-pico/blob/master/examples/unix/c11/z_get.c) and [queryable](https://github.com/eclipse-zenoh/zenoh-pico/blob/master/examples/unix/c11/z_queryable.c)). As a reminder, a query has now gained the possibility to carry some user payload that can be received and interpreted by the matching queryables. For example, you can now attach a picture to a query that is analyzed by a queryable running an object detection algorithm as shown in the figure below.

{{< figure-inline
    src="../../img/20230503-blog-zenoh-charmander2/zenoh-queryable-example.png"
    class="figure-inline"
    alt="Query payload"
    width="60%" >}}

## TLS updates

This release adds the support for [Let’s Encrypt](https://letsencrypt.org/) TLS certificates in Zenoh. More information on how to use it, for example to set up a Zenoh router in the cloud with TLS support can be found in this [dedicated blog post](https://zenoh.io/blog/2023-04-04-letsencrypt/).

Moreover, the TLS library used by Zenoh ([Rustls](https://www.memorysafety.org/blog/rustls-new-features/)) has finally added the support for TLS certificates based on IP addresses instead of DNS names. To test it out, it is enough to generate the TLS certificates according to [these instructions](https://zenoh.io/docs/manual/tls/#appendix-tls-certificates-creation).

{{< figure-inline
    src="../../img/20230503-blog-zenoh-charmander2/zenoh-letsencrypt.png"
    class="figure-inline"
    alt="Letsencrypt validation"
    width="60%" >}}

## S3 Backend updates

This release also includes some updates for the S3 backend. The first version of the plugin already provided many functionalities, such as the communication between Zenoh and AWS S3 servers, compatibility with MinIO S3 instances, and TLS communication. However we were missing timestamp handling on our side, which could lead to some edge case concurrency issues. In this release we handle this problem.

## Key Expression Trees and Formatters

With Zenoh 0.6, we introduced a new definition for Key Expressions (KE) to make them both faster and more predictable. Now introducing KeTrees and KeFormatters (Rust exclusive for now). A full blog post on both of these is coming soon, but in the meantime, here’s a quick sum-up.

KeTrees: it may be tempting to just put KE-value pairs into a hashmap and be done with it, but the reality is that you’re likely to lose the intersection/inclusion semantics of KEs when doing that. KeTrees are built to let you iterate on intersections between KEs as fast as possible.

```rust
let mut tree = KeBoxTree::new();
tree.insert(keyexpr::new("a/b").unwrap(), 1);
tree.insert(keyexpr::new("a/c").unwrap(), 2);
tree.insert(keyexpr::new("b/c").unwrap(), 3);
let intersection = tree.intersection(keyexpr::new("a/**").unwrap());
// intersection is an iterator which will yield the nodes for "a",
// "a/b" and "a/c", which will have weights `None`, `Some(1)` and `Some(2)`
// respectively. The order isn't guaranteed.
```

KeFormatters: because KEs are Zenoh’s address-space, they’re a fairly important thing for your team to manage correctly. You’re likely to end up defining your address space in a manner similar to REST APIs. KeFormatters are here to help you manage this in 3 steps:

- Define a format: `zenoh::kedefine!(pub temperatures: "factory/temperature/sensor/-/${factory:*}/${sensor:*}");`
- Use it to construct KEs, letting the formatter ensure that you aren’t accidentally breaking your API (setting `factory = “5/42”` in the following example, for example, would cause an error.

```rust
let mut formatter = temperatures::formatter();
let ke = zenoh::keformat!(formatter, factory = 5, sensor = 42).unwrap();
```

- Or use it to parse incoming KEs to extract the informations you actually care about:

```rust
let parsed = temperatures::parse(ke).unwrap();
assert_eq!(parsed.factory(), Ok("5"));
assert_eq!(parsed.sensor(), Ok("42"));
```

## Transport protocol whitelisting

A new configuration option has been added in zenoh to allow only certain network protocols to be used by Zenoh. In order to understand what that means, we need to take a step back and see how Zenoh operates. Zenoh can work on various network protocols as shown in the figure below.

{{< figure-inline
    src="../../img/20230503-blog-zenoh-charmander2/zenoh-stack.png"
    class="figure-inline"
    alt="Zenoh stack"
    width="40%" >}}

Depending on the use case and requirements, certain protocols may be preferred to others. E.g., we would like to force all the Zenoh communication to happen on TLS, which provides encryption, and not on TCP, which is less secure. In normal operation, Zenoh will try to connect via any available means to the other Zenoh nodes (e.g. peer or router). I.e., if a node supports multiple protocols (e.g. TCP, UDP, TLS), it will try any of those until it succeeds. However, in some cases it would be nice to limit the protocols to be used. And how to do that? With the transport protocol whitelisting introduced in this release! The figure below shows an example of a Zenoh peer who has configured only TLS as protocol whitelist and it will fail to connect to a Zenoh peer only offering TCP connectivity.

{{< figure-inline
    src="../../img/20230503-blog-zenoh-charmander2/zenoh-whitelist-ip.png"
    class="figure-inline"
    alt="IP whitelisting"
    width="60%" >}}

Here below is an example of the configuration for the transport whitelisting.

```
transport: {
    link: {
        /// An optional whitelist of protocols to be used for accepting and opening sessions.
        /// If not configured, all the supported protocols are automatically whitelisted.
        /// The supported protocols are: ["tcp" , "udp", "tls", "quic", "ws", "unixsock-stream"]
        /// For example, to only enable "tls" and "quic":
        protocols: ["tls", "quic"],
    }
}
```

## ROS1 bridge

[ROS1](https://wiki.ros.org) is a very popular meta-operating system for robotics. It consists of the main core (peer-to-peer message exchange primitives, environment, broker - “rosmaster”, interface definition tools, package system) and numerous packages and tools providing extra capabilities built on top of the core: HAL, device drivers, components and algorithms collection etc.

A typical ROS1 system is a set of services communicating through some closed network and performing on different levels: from low-level hardware control (servo motors, sensors) and up to some high-level functionality (autopiloting, telemetry collection, parameter tuning, remote control etc).

Despite the fact that ROS1 operates on a peer-to-peer network, it is not designed to be ultimately scalable and leaves a lot of scalability problems to be solved by the user. In this condition, we believe that bridging ROS1 systems to Zenoh could do the trick, allowing ROS1 users to utilize all power of Zenoh in their solutions.

An alpha version of ROS1 to Zenoh bridge is introduced in this release. This bridge is quite similar to the ROS2 bridge, but it offers some limited functionality for ROS1 systems. An example integration is shown on the following schema:

{{< figure-inline
    src="../../img/20230503-blog-zenoh-charmander2/zenoh-ros1.png"
    class="figure-inline"
    alt="Zenoh & Ros1"
    width="60%" >}}

Capabilities:

- Bridging publishers, subscribers, services, clients
- ROS1-to-ROS1 interaction support
- ROS1-to-Zenoh interaction support

ROS1 bridge has some limitations which, however, are planned to be completely eliminated in the nearest future:

- All topic names are bridged as-is
- All topic data types and md5 sums are bridged as "\*" wildcard and may not work with some ROS1 client implementations
- Performance may be significantly improved, mainly by improving the Rust ROS1 bindings.
- Supports only TCPROS protocol

## Liveliness support

This release introduces the _liveliness_ feature. It allows any Zenoh application to assert and monitor the liveliness of any other Zenoh application in the network. Zenoh applications can declare liveliness tokens associated with some key expressions. Liveliness tokens will be seen as _alive_ by other Zenoh applications while the Zenoh application that declared it is _alive_.

Zenoh applications can query alive tokens with the `get_liveliness` function and subscribe to liveliness changes (apparition/disappearance of liveliness tokens) with the `declare_liveliness_subscriber` function.
This feature is only available in the Rust API and is marked _unstable_: it works as intended but the API may change in the future. More details can be found [here](https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Liveliness.md). Examples can be found [here](https://github.com/eclipse-zenoh/zenoh/blob/master/examples/examples/z_liveliness.rs), [here](https://github.com/eclipse-zenoh/zenoh/blob/master/examples/examples/z_get_liveliness.rs) and [here](https://github.com/eclipse-zenoh/zenoh/blob/master/examples/examples/z_sub_liveliness.rs).
