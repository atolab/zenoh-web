---
title: "TLS authentication"
weight : 3600
menu:
  docs:
    parent: manual
---

Zenoh supports TLS as a transport protocol.
TLS can be configured in two ways:
* server side authentication: clients validate the server TLS certificate but not the other way around, that is, the same way of operating on the web where the web browsers validate the identity of the server via means of the TLS certificate.

* mutual authentication (mTLS): where both server-side and client-side authentication is required.

The configuration of TLS certificates is done via a [configuration file](../configuration.md).

---------


## Client configuration

The field **root_ca_certificate** is used to specify the path to the certificate used to authenticate the *TLS server*. 

It's important to note that if the field is not specified then the default behaviour is to load the root certificates provided by [Mozilla's CA for use with webpki](https://docs.rs/crate/webpki-roots/latest/source/src/lib.rs).

However, if we manage our own certificates, we need to specify the root certificate. 
Suppose we generated the certificate using [MiniCA as explained below](#appendix-tls-certificates-creation), then the configuration file for a *client* would be:
```json
{
  /// The node's mode (router, peer or client)
  mode: "client",
  connect: {
    endpoints: [ "tls/localhost:7447" ]
  },
  transport: {
    link: {
      tls: {
        root_ca_certificate: "/home/user/tls/minica.pem",
      },
    },
  },
}
```

When using such configuration, the client will use the provided `minica.pem` certificate to authenticate the *TLS server certificate*.

Let's assume the above configuration is then saved with the name *client.json5*.

## Router configuration

The required **tls** fields for configuring a *TLS certificate* for a router are **server_private_key** and **server_certificate**.

A configuration file for a *router* would be:
```json
{
  /// The node's mode (router, peer or client)
  mode: "router",
  listen: {
    endpoints: [ "tls/localhost:7447" ]
  },
  transport: {
    link: {
      tls: {
        server_private_key: "/home/user/tls/localhost/key.pem",
        server_certificate: "/home/user/tls/localhost/cert.pem",
      },
    },
  },
}
```

When using such configuration, the router will use the provided **server_private_key** and **server_certificate** for establishing a TLS session with any client.

Let's assume that the above configurations are then saved with the name *server.json5*.

## Peer configuration

The required **tls** fields for configuring a *TLS certificate* for a router are **root_ca_certificate**, **server_private_key** and **server_certificate**.

A configuration file for a *peer* would be:
```json
{
  /// The node's mode (router, peer or client)
  mode: "peer",
  transport: {
    link: {
      tls: {
        root_ca_certificate: "/home/user/tls/minica.pem",
        server_private_key: "/home/user/tls/localhost/key.pem",
        server_certificate: "/home/user/tls/localhost/cert.pem",
      },
    },
  },
}
```

When using such configuration, the peer will use the provided **root_ca_certificate** to authenticate the *TLS certificate* of the *peer* it is connecting to.
At the same time, the peer will use the provided **server_private_key** and **server_certificate** for initiating incoming TLS sessions from other peers.

Let's assume that the above configurations are then saved with the name *peer.json5*.

---------
## Mutual authentication (mTLS)

In order to enable mutual authentication, we'll need two sets of keys and certificates, one for the "server" and one for the "client". These sets of keys and certificates can be generated as explained [in the appendix section below](#appendix-tls-certificates-creation).
Let's suppose we are storing them under `$home/user/` with the following files and folders structure:
```bash
user
├── client
│   ├── localhost
│   │   ├── cert.pem
│   │   └── key.pem
│   ├── minica-key.pem
│   └── minica.pem
└── server
    ├── localhost
    │   ├── cert.pem
    │   └── key.pem
    ├── minica-key.pem
    └── minica.pem
```

### Router configuration

The filed `client_auth` needs to be set to `true` and we must provide the router (acting as server) the certificate authority to validate the client's keys and certificates under the field `root_ca_certificate`. The `server_private_key` and `server_certificate` fields are also required in order to authenticate the router in front of the client.

```json
{
  mode: "router",
  listen: {
    endpoints: [ "tls/localhost:7447" ]
  },
  transport: {
    link: {
      tls: {
        root_ca_certificate: "/home/user/client/minica.pem",
        client_auth: true,
        server_private_key: "/home/user/server/localhost/key.pem",
        server_certificate: "/home/user/server/localhost/cert.pem",
      },
    },
  },
}
```

### Client configuration

Again, the field `client_auth` needs to be set to `true` and we must provide the certificate authority to validate the server keys and certificates. Similarly, we need to provide the client keys and certificates for the server to authenticate our connection.
```json
{
  mode: "client",
  connect: {
    endpoints: [ "tls/localhost:7447" ]
  },
  transport: {
    link: {
      tls: {
        root_ca_certificate: "/home/user/server/minica.pem",
        client_auth: true,
        client_private_key: "/home/user/client/localhost/key.pem",
        client_certificate: "/home/user/client/localhost/cert.pem",
      },
    },
  },
}
```

---------
## Testing the TLS transport

Let's assume a scenario with one Zenoh router and two clients connected to it: one publisher and one subscriber.

The first thing to do is to run the router passing its configuration, i.e. *router.json5*:
```bash
$ zenohd -c router.json5
```

Then, let's start the subscriber in client mode passing its configuration, i.e. *client.json5*:
```bash
$ z_sub -c client.json5
```

Lastly, let's start the publisher in client mode passing its configuration, i.e. *client.json5*:
```bash
$ z_pub -c client.json5
```

As it can be noticed, the same *client.json5* is used for *z_sub* and *z_pub*.

### Peer-to-peer scenario
Let's assume a scenario with two peers.

First, let's start the first peer in peer mode passing its configuration, i.e. *peer.json5*:
```bash
$ zn_sub -c peer.json5 -l tls/localhost:7447
```

Next, let's start the second peer in peer mode passing its configuration, i.e. *peer.json5*:
```bash
$ zn_pub -c peer.json5 -l tls/localhost:7448 -e tls/localhost:7447
```

As it can be noticed, the same *peer.json5* is used for *zn_sub* and *zn_pub*.

---------
## Appendix: TLS certificates creation

In order to use TLS as a transport protocol, we need first to create the TLS certificates. 
While multiple ways of creating TLS certificates exist, in this guide we are going to use [minica](https://github.com/jsha/minica) for simplicity:

> *Minica is a simple CA intended for use in situations where the CA operator also operates each host where a certificate will be used. It automatically generates both a key and a certificate when asked to produce a certificate. It does not offer OCSP or CRL services. Minica is appropriate, for instance, for generating certificates for RPC systems or microservices.*

First, you need to install minica by following these [instructions](https://github.com/jsha/minica#installation).
Once you have successfully installed on your machine, let's create the certificates as follows assuming that we will test Zenoh over TLS on *localhost*.
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

Once the above certificates have been correctly generated, we can proceed to configure Zenoh to use TLS as explained.
