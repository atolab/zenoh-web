---
title: "QUIC transport"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports QUIC as a transport protocol.

As you may already know, QUIC is a UDP-based, stream-multiplexing, encrypted transport protocol.
It natively embeds TLS for encryption, authentication and confidentiality.

As of today, the only supported TLS authentication mode in zenoh is server-authentication: clients validate the server TLS certificate but not the other way around.
That is, the same way of operating on the web where the web browsers validate the identity of the server via means of the TLS certificate.

---------
## TLS configuration

In order to use QUIC as a transport protocol, we need first to create the TLS certificates. 

The instructions to properly generate TLS certificates can be found [here](./tls). 

As you can see, they are the same instructions required to run zenoh on TLS over TCP. 
Here instead, the only difference is that we have TLS in QUIC!
Nevertheless, the procedures to generate the certificates are exactly the same.

---------
## Testing the QUIC transport

You can test out zenoh over QUIC in both client-router and peer-to-peer scenarios.

### Client-Router scenario
Let's assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.
The first thing to do is to generate the *router.json5* and *client.json5* configuration files as explained [here](./tls).

Next, it's time to run the router passing its configuration, i.e. *router.json5*:
```bash
$ zenohd -c router.json5
```

Then, let's start the subscriber in client mode passing its configuration, i.e. *client.json5*:
```bash
$ zn_sub -c client.json5
```

Lastly, let's start the publisher in client mode passing its configuration, i.e. *client.json5*:
```bash
$ zn_pub -c client.json5
```

As it can be noticed, the same *client.json5* is used for *zn_sub* and *zn_pub*.

### Peer-to-peer scenario
Let's assume a scenario with two peers.
The first thing to do is to generate the *peer.json5* configuration files as explained [here](./tls).

Then, let's start the first peer in peer mode passing its configuration, i.e. *peer.json5*:
```bash
$ zn_sub -c peer.json5 -l quic/localhost:7447
```

Lastly, let's start the second peer in peer mode passing its configuration, i.e. *peer.json5*:
```bash
$ zn_pub -c peer.json5 -l quic/localhost:7448 -e quic/localhost:7447
```

As it can be noticed, the same *peer.json5* is used for *zn_sub* and *zn_pub*.
