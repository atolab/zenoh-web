---
title: "TLS authentication"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports TLS as a transport protocol.

Currently, the only supported TLS authentication mode is server-authentication: clients validate the server TLS certificate but not the other way around.
This is the same way the web operates, where the web browsers validate the identity of the server via means of the TLS certificate.

---------
## TLS certificates creation

In order to use TLS as a transport protocol, you need to create the TLS certificates. There are multiple ways to create TLS certificates, this guide uses [minica](https://github.com/jsha/minica):

> *Minica is a simple CA intended for use in situations where the CA operator also operates each host where a certificate will be used. It automatically generates both a key and a certificate when asked to produce a certificate. It does not offer OCSP or CRL services. Minica is appropriate, for instance, for generating certificates for RPC systems or microservices.*

Install minica by following the [minica installation instructions](https://github.com/jsha/minica#installation).
Once you have successfully installed minica, create the certificates as follows, this assumes you will test zenoh over TLS on *localhost*.
Create a folder to store the certificates:

```bash
$/home/user: mkdir tls
$/home/user: cd tls
$/home/user/tls: pwd
/home/user/tls
```

Generate the TLS certificate for the *localhost* domain:
```bash
$/home/user/tls: minica --domains localhost
```

This creates the following files:
```bash
$/home/user/tls: ls
localhost   minica-key.pem  minica.pem
```

*minica.pem* is the root CA certificate that the client uses to validate the server certificate.
The server certificate *cert.pem* and private key *key.pem* are in the *localhost* folder.

```bash
$/home/user/tls: ls localhost
cert.pem    key.pem
```

Once the certificates have been generated, configure zenoh to use TLS.

---------
## Client configuration

The [zenoh property](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for authenticating a *TLS server* for a client is **tls_root_ca_certificate**.

A configuration file for a *client* is as follows:

```
tls_root_ca_certificate=/home/user/tls/minica.pem
```

With this configuration, the client uses the **tls_root_ca_certificate** to authenticate the *TLS server certificate*.

We assume the above configuration is saved with the name *client.conf*.

## Router configuration

The [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for configuring a *TLS certificate* for a router are **tls_server_private_key** and **tls_server_certificate**.

A configuration file for a *router* is as follows:
```
tls_server_private_key=/home/user/tls/localhost/key.pem
tls_server_certificate=/home/user/tls/localhost/cert.pem
```

With this configuration, the router uses the **tls_server_private_key** and **tls_server_certificate** to establish a TLS session with any client.

We assume the above configurations are saved with the name *server.conf*.

## Peer configuration

The [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for configuring a *TLS certificate* for a peer are **tls_root_ca_certificate**, **tls_server_private_key** and **tls_server_certificate**.

A configuration file for a *peer* is as follows:
```
tls_root_ca_certificate=/home/user/tls/minica.pem
tls_server_private_key=/home/user/tls/localhost/key.pem
tls_server_certificate=/home/user/tls/localhost/cert.pem
```

With this configuration, the peer uses the **tls_root_ca_certificate** to authenticate the *TLS certificate* of the *peer* it is connecting to. At the same time, the peer uses the **tls_server_private_key** and **tls_server_certificate** for initiating incoming TLS sessions from other peers.

We assume that the above configurations are saved with the name *peer.conf*.

---------
## Test the TLS transport

We assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.

Run the router passing its configuration, i.e. *router.conf*:
```bash
$ zenohd -c router.conf -l tls/localhost:7447
```

Start the subscriber in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_sub --mode='client' -c client.conf -e tls/localhost:7447
```

Start the publisher in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_pub --mode='client' -c client.conf -e tls/localhost:7447
```

The same *client.conf* is used for *zn_sub* and *zn_pub*.
