---
title: "QUIC transport"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports QUIC as a transport protocol.

QUIC is a UDP-based, stream-multiplexing, encrypted transport protocol. It natively embeds TLS for encryption, authentication and confidentiality.

As of today, the only supported TLS authentication mode in zenoh is server-authentication: clients validate the server TLS certificate but not the other way around. This is, the same way the web operates where web browsers validate the identity of the server via the TLS certificate.

---------
## TLS configuration

In order to use QUIC as a transport protocol, we first need to create the TLS certificates. Refer to [generate TLS certificates](../tls). 

These are the same instructions as is required to run zenoh on TLS over TCP, the only difference is that we have TLS in QUIC. The procedures to generate the certificates are exactly the same.

---------
## Test the QUIC transport

You can test zenoh over QUIC in both client-router and peer-to-peer scenarios.

### Client-router scenario
Let's assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.
First generate the *router.conf* and *client.conf* configuration files as explained within [TLS authentication](../tls).

Run the router passing its configuration, i.e. *router.conf*:
```bash
$ zenohd -c router.conf -l quic/localhost:7447
```

Start the subscriber in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_sub --mode='client' -c client.conf -e quic/localhost:7447
```

Start the publisher in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_pub --mode='client' -c client.conf -e quic/localhost:7447
```

The same *client.conf* is used for *zn_sub* and *zn_pub*.

### Peer-to-peer scenario
Let's assume a scenario with two peers.
First generate the *peer.conf* configuration files as explained within [TLS authentication](../tls).

Start the first peer in peer mode passing its configuration, i.e. *peer.conf*:
```bash
$ zn_sub --mode='peer' -c peer.conf -e quic/localhost:7447
```

Start the second peer in peer mode passing its configuration, i.e. *peer.conf*:
```bash
$ zn_pub --mode='peer' -c peer.conf -e quic/localhost:7447
```

The same *peer.conf* is used for *zn_sub* and *zn_pub*.
