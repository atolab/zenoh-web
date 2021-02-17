---
title: "TLS authentication"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports TLS as a transport protocol.
As of today, the only supported TLS authentication mode is server-authentication: clients validate the server TLS certificate but not the other way around.
That is, the same way of operating in the web where the web browsers validate the identity of the server via means of the TLS certificate.

---------
## TLS certificates creation

In order to use TLS as a transport protocol, we need first to create the TLS certificates. 
While multiple ways of creating TLS certificates exist, in this guide we are going to use [minica](https://github.com/jsha/minica) for simplicity:

> *Minica is a simple CA intended for use in situations where the CA operator also operates each host where a certificate will be used. It automatically generates both a key and a certificate when asked to produce a certificate. It does not offer OCSP or CRL services.  Minica is appropriate, for instance, for generating certificates for RPC systems or microservices.*

First of all, you need to install minica by following these [instructions](https://github.com/jsha/minica#installation).
Once you have succesfully installed on your machine, let's create the certificates as follows assuming that we will test zenoh over TLS on *localhost*.
First let's create a folder to store our certificates:
```bash
$/home/user: mkdir tls
$/home/user: cd tls
$/home/user/tls: pwd
/home/user/tls
```

Then, let's generate the TLS certificate for the *localhost* domain:
```bash
$/home/user/tls: minica --domains localhost
```

This should create the following files:
```bash
$/home/user/tls: ls
localhost   minica-key.pem  minica.pem
```

*minica.pem* is the root CA certificate that will be used by the client to validate the server certificate.
The server certificate *cert.pem* and private key *key.pem* can be found inside the *localhost* folder.
```bash
$/home/user/tls: ls localhost
cert.pem    key.pem
```

Once the above certificates have been correctly generated, we can proceed to configure zenoh to use TLS.

---------
## Client configuration

The required [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for authenticating a *TLS server* for a client is **tls_root_ca_certificate**.
A configuration file for a *client* would be:
```
tls_root_ca_certificate=/home/user/tls/minica.pem
```

When using such configuration, the client will use the provided **tls_root_ca_certificate** to authenticate the *TLS server certificate*.

Let's assume the above configuration is then saved with the name *client.conf*.

## Router configuration

The required [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for configuring a *TLS certificate* for a router are **tls_server_private_key** and **tls_server_certificate**.

A configuration file for a *router* would be:
```
tls_server_private_key=/home/user/tls/localhost/key.pem
tls_server_certificate=/home/user/tls/localhost/cert.pem
```

When using such configuration, the router will use the provided **tls_server_private_key** and **tls_server_certificate** for establishing a TLS session with any client.

Let's assume that the above configurations are then saved with the name *server.conf*.

## Peer configuration

The required [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for configuring a *TLS certificate* for a router are **tls_root_ca_certificate**, **tls_server_private_key** and **tls_server_certificate**.

A configuration file for a *peer* would be:
```
tls_root_ca_certificate=/home/user/tls/minica.pem
tls_server_private_key=/home/user/tls/localhost/key.pem
tls_server_certificate=/home/user/tls/localhost/cert.pem
```

When using such configuration, the peer will use the provided **tls_root_ca_certificate** to authenticate the *TLS certificate* of the *peer* he is connecting to.
At the same time, the peer will use the provided **tls_server_private_key** and **tls_server_certificate** for initiating ingoming TLS sessions from other peers.

Let's assume that the above configurations are then saved with the name *peer.conf*.

---------
## Testing the TLS transport

Let's assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.

The first thing to do is to run the router passing its configuration, i.e. *router.conf*:
```bash
$ zenohd -c router.conf -l tls/localhost:7447
```

Then, let's start the subscriber in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_sub --mode='client' -c client.conf -e tls/localhost:7447
```

Lastly, let's start the publisher in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_pub --mode='client' -c client.conf -e tls/localhost:7447
```

As it can be noticed, the same *client.conf* is used for *zn_sub* and *zn_pub*.
