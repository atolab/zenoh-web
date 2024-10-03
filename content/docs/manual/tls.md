---
title: "TLS authentication"
weight: 3600
menu:
  docs:
    parent: manual
---

Zenoh supports TLS as a transport protocol.
TLS can be configured in two ways:

- server side authentication: clients validate the server TLS certificate but not the other way around, that is, the same way of operating on the web where the web browsers validate the identity of the server via means of the TLS certificate.

- mutual authentication (mTLS): where both server-side and client-side authentication is required.

The configuration of TLS certificates is done via a [configuration file](../configuration).

---

## Client configuration

The field **root_ca_certificate** is used to specify the path to the certificate used to authenticate the _TLS server_.

It's important to note that if the field is not specified then the default behaviour is to load the root certificates provided by [Mozilla's CA for use with webpki](https://docs.rs/crate/webpki-roots/latest/source/src/lib.rs).

However, if we manage our own certificates, we need to specify the root certificate.
Suppose we generated the certificate using [MiniCA as explained below](#appendix-tls-certificates-creation), then the configuration file for a _client_ would be:

```json
{
  /// The node's mode (router, peer or client)
  "mode": "client",
  "connect": {
    "endpoints": ["tls/localhost:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "root_ca_certificate": "/home/user/tls/minica.pem"
      }
    }
  }
}
```

When using such configuration, the client will use the provided `minica.pem` certificate to authenticate the _TLS server certificate_.

Let's assume the above configuration is then saved with the name _client.json5_.

## Router configuration

The required **tls** fields for configuring a _TLS certificate_ for a router are **listen_private_key** and **listen_certificate**.

A configuration file for a _router_ would be:

```json
{
  /// The node's mode (router, peer or client)
  "mode": "router",
  "listen": {
    "endpoints": ["tls/localhost:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "listen_private_key": "/home/user/tls/localhost/key.pem",
        "listen_certificate": "/home/user/tls/localhost/cert.pem"
      }
    }
  }
}
```

When using such configuration, the router will use the provided **listen_private_key** and **listen_certificate** for establishing a TLS session with any client.

Let's assume that the above configurations are then saved with the name _server.json5_.

## Peer configuration

The required **tls** fields for configuring a _TLS certificate_ for a peer are **root_ca_certificate**, **listen_private_key** and **listen_certificate**.

A configuration file for a _peer_ would be:

```json
{
  /// The node's mode (router, peer or client)
  "mode": "peer",
  "transport": {
    "link": {
      "tls": {
        "root_ca_certificate": "/home/user/tls/minica.pem",
        "listen_private_key": "/home/user/tls/localhost/key.pem",
        "listen_certificate": "/home/user/tls/localhost/cert.pem"
      }
    }
  }
}
```

When using such configuration, the peer will use the provided **root_ca_certificate** to authenticate the _TLS certificate_ of the _peer_ it is connecting to.
At the same time, the peer will use the provided **listen_private_key** and **listen_certificate** for initiating incoming TLS sessions from other peers.

Let's assume that the above configurations are then saved with the name _peer.json5_.

---

## TLS with Scouting :warning:

Zenoh provides a [scouting mechanism](../../getting-started/deployment/#scouting) that allows peers to discover other neighboring peers automatically.

By default, this feature is enabled and attempts to establish connections with other peers **using all Zenoh-supported protocols** (not just TLS).

To ensure that all connections are established using TLS, you can configure the protocols filter as shown below:

```json
{
  "transport": {
    "link": {
      "protocols": ["tls"]
    }
  }
}
```

The `protocols` configuration field specifies which protocols Zenoh should whitelist for accepting and opening sessions. If this field is not configured, Zenoh will automatically whitelist all supported protocols.

---

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

The filed `enable_mtls` needs to be set to `true` and we must provide the router (acting as server) the certificate authority to validate the client's keys and certificates under the field `root_ca_certificate`. The `listen_private_key` and `listen_certificate` fields are also required in order to authenticate the router in front of the client.

```json
{
  "mode": "router",
  "listen": {
    "endpoints": ["tls/localhost:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "root_ca_certificate": "/home/user/client/minica.pem",
        "enable_mtls": true,
        "listen_private_key": "/home/user/server/localhost/key.pem",
        "listen_certificate": "/home/user/server/localhost/cert.pem"
      }
    }
  }
}
```

### Client configuration

Again, the field `enable_mtls` needs to be set to `true` and we must provide the certificate authority to validate the server keys and certificates. Similarly, we need to provide the client keys and certificates for the server to authenticate our connection.

```json
{
  "mode": "client",
  "connect": {
    "endpoints": ["tls/localhost:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "root_ca_certificate": "/home/user/server/minica.pem",
        "enable_mtls": true,
        "connect_private_key": "/home/user/client/localhost/key.pem",
        "connect_certificate": "/home/user/client/localhost/cert.pem"
      }
    }
  }
}
```

---

## Testing the TLS transport

Let's assume a scenario with one Zenoh router and two clients connected to it: one publisher and one subscriber.

The first thing to do is to run the router passing its configuration, i.e. _router.json5_:

```bash
$ zenohd -c router.json5
```

Then, let's start the subscriber in client mode passing its configuration, i.e. _client.json5_:

```bash
$ z_sub -c client.json5
```

Lastly, let's start the publisher in client mode passing its configuration, i.e. _client.json5_:

```bash
$ z_pub -c client.json5
```

As it can be noticed, the same _client.json5_ is used for _z_sub_ and _z_pub_.

### Peer-to-peer scenario

Let's assume a scenario with two peers.

First, let's start the first peer in peer mode passing its configuration, i.e. _peer.json5_:

```bash
$ z_sub -c peer.json5 -l tls/localhost:7447
```

Next, let's start the second peer in peer mode passing its configuration, i.e. _peer.json5_:

```bash
$ z_pub -c peer.json5 -l tls/localhost:7448 -e tls/localhost:7447
```

As it can be noticed, the same _peer.json5_ is used for _z_sub_ and _z_pub_.

---

## Appendix: TLS certificates creation

In order to use TLS as a transport protocol, we need first to create the TLS certificates.
While multiple ways of creating TLS certificates exist, in this guide we are going to use [minica](https://github.com/jsha/minica) for simplicity:

> _Minica is a simple CA intended for use in situations where the CA operator also operates each host where a certificate will be used. It automatically generates both a key and a certificate when asked to produce a certificate. It does not offer OCSP or CRL services. Minica is appropriate, for instance, for generating certificates for RPC systems or microservices._

First, you need to install minica by following these [instructions](https://github.com/jsha/minica#installation).
Once you have successfully installed on your machine, let's create the certificates as follows assuming that we will test Zenoh over TLS on _localhost_.
First let's create a folder to store our certificates:

```bash
$/home/user: mkdir tls
$/home/user: cd tls
$/home/user/tls: pwd
/home/user/tls
```

Then, let's generate the TLS certificate for the _localhost_ domain:

```bash
$/home/user/tls: minica --domains localhost
```

This should create the following files:

```bash
$/home/user/tls: ls
localhost   minica-key.pem  minica.pem
```

_minica.pem_ is the root CA certificate that will be used by the client to validate the server certificate.
The server certificate _cert.pem_ and private key _key.pem_ can be found inside the _localhost_ folder.

```bash
$/home/user/tls: ls localhost
cert.pem    key.pem
```

Once the above certificates have been correctly generated, we can proceed to configure Zenoh to use TLS as explained.

---

Since version 0.7.1-rc from Zenoh, we can generate certificates associated not only to dns domains but also to ip addresses as well. For instance, we can generate them as follows with minica:

```bash
$/home/user/server/tls: minica --ip-addresses 127.0.0.1
```

```bash
$/home/user/server/tls: ls
127.0.0.1   minica-key.pem  minica.pem
```

Then on the Zenoh configuration file we'll be able to set up the TLS configuration specifying the ip address, for instance for a server and a client with tls:

```json
{
  "mode": "router",
  "listen": {
    "endpoints": ["tls/127.0.0.1:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "listen_private_key": "/home/user/server/127.0.0.1/key.pem",
        "listen_certificate": "/home/user/server/127.0.0.1/cert.pem"
      }
    }
  }
}
```

```json
{
  "mode": "client",
  "connect": {
    "endpoints": ["tls/127.0.0.1:7447"]
  },
  "transport": {
    "link": {
      "tls": {
        "root_ca_certificate": "/home/user/server/minica.pem"
      }
    }
  }
}
```
