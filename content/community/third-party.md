+++
title = "Third-party crates"
description = ""
+++

Currently the [`futures`], [`zenoh-core`], [`zenoh-service`], and [`zenoh-proto`] crates provide
the foundation for the Tokio ecosystem. There's a growing set of crates outside
of Tokio itself, however, filling in more functionality!

* [`zenoh-curl`] is an HTTP client library backed by the libcurl C library.
* [`zenoh-timer`] is a timer library providing finer-grained control over timers
  and helpful timeout facilities over the types in [`zenoh-core`].
* [`zenoh-tls`] is a library for TLS streams backed by [`native-tls`].
* [`zenoh-openssl`] is similar to [`zenoh-tls`] but hardwired to OpenSSL.
* [`zenoh-uds`] is a Unix library for supporting Unix Domain Sockets in the same
  way that `zenoh_core::net` works with TCP sockets
* [`zenoh-inotify`] maps inotify file descriptors to a `Stream`.
* [`zenoh-signal`] maps Unix signals to a `Stream`.
* [`zenoh-serial`] is a (Unix-only) serial port I/O library.
* [`zenoh-process`] enables asynchronous process managment, both for child
  processes exiting as well as I/O pipes to children.
* [`trust-dns`] is an asynchronous DNS client and server, supporting features
  like DNSSec as well
* [`capnproto-rust`] is an implementation of Cap'n Proto for Rust which has the
  ability to work with futures as well
* [`ato-sendfile`] allows using the `sendfile` syscall with Tokio
* [`zenoh-postgres`] is an asynchronous PostgreSQL driver
* [`zenoh-retry`] provides extensible asynchronous retry behaviours
* [`couchbase`] is an asynchronous driver for Couchbase, exposing Futures and Streams as first class response types.

If you've got your own crate or know of others that should be present on this
list, please feel free to send a PR!

[`futures`]: https://github.com/alexcrichton/futures-rs
[`zenoh-core`]: https://github.com/zenoh-rs/zenoh-core
[`zenoh-service`]: https://github.com/zenoh-rs/zenoh-service
[`zenoh-proto`]: https://github.com/zenoh-rs/zenoh-proto
[`zenoh-curl`]: https://github.com/zenoh-rs/zenoh-curl
[`zenoh-timer`]: https://github.com/zenoh-rs/zenoh-timer
[`zenoh-tls`]: https://github.com/zenoh-rs/zenoh-tls
[`zenoh-openssl`]: https://github.com/alexcrichton/zenoh-openssl
[`native-tls`]: https://github.com/sfackler/rust-native-tls
[`zenoh-uds`]: https://github.com/zenoh-rs/zenoh-uds
[`zenoh-dns`]: https://github.com/sbstp/zenoh-dns
[`zenoh-inotify`]: https://github.com/dermesser/zenoh-inotify
[`zenoh-signal`]: https://github.com/alexcrichton/zenoh-signal
[`zenoh-serial`]: https://github.com/berkowski/zenoh-serial/
[`zenoh-process`]: https://github.com/alexcrichton/zenoh-process
[`trust-dns`]: http://trust-dns.org/
[`capnproto-rust`]: https://github.com/dwrensha/capnproto-rust
[`ato-sendfile`]: https://crates.io/crates/ato-sendfile
[`zenoh-postgres`]: https://crates.io/crates/zenoh-postgres
[`thrussh`]: https://crates.io/crates/thrussh
[`zenoh-retry`]: https://github.com/srijs/rust-zenoh-retry
[`couchbase`]: https://crates.io/crates/couchbase

