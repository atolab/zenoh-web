---
title: "User-Password authentication"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports basic *user-password authentication*.

Clients and peers can use *user* and *password* for authentication against a router or a peer. The configuration of credentials is done via a configuration file defining certain [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html).

---------
## Client configuration

The [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for basic *user-password authentication* for a client are **user** and **password**.

A configuration file for a *client* is as follows:

```
user=clientusername
password=clientpassword
```

With this configuration, the client uses the **user** and **password** to authenticate against any peer or router.

We assume the above configuration is saved with the name *client.conf*.

## Router or peer configuration

The [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for basic *user-password authentication* for a client are **user**, **password**, **user_password_dictionary**. The **user_password_dictionary** zenoh property indicates the path of a text file containing the full list of authorized credentials for connections.

A configuration file for a *router* or *peer* is as follows:
```
user=routerusername
password=routerpassword
user_password_dictionary=credentials.conf
```

The authorized credentials text file is:
```
clientusername:clientpassword
```
Each line of the file represents the one single authorized credential in the form of *user:password*. The authorized credentials text file can contain any number of lines.

With this configuration, the router or peer uses the **user** and **password** to authenticate against any other peer or router. At the same time, it uses the file containing the authorized credentials to authenticate incoming connections.

We assume that the above configurations are saved with the name *router.conf* and *credentials.conf*.


---------
## Test the user-password authentication

We assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.

Run the router passing its configuration, i.e. *router.conf*:
```bash
$ zenohd -c router.conf
```
Ensure that the path indicated in the *user_password_dictionary* property points to a valid credentials file.

Start the subscriber in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_sub --mode='client' -c client.conf
```

Start the publisher in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_pub --mode='client' -c client.conf
```

The same *client.conf* is used for *zn_sub* and *zn_put*. Both are using the same credentials and are authenticated accordingly by the *router*. However, different configuration files and credentials could be used.

---------
## Hash passwords

If you do not want to store clear text passwords in the configuration files, before you create the configuration files, use a hashing function to hash the password. 
For example, let's compute the hash for the *client* and *router* passwords:

```bash
$ echo clientpassword | sha256sum
92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724

$ echo routerpassword | sha256sum
2ce01a11893f276a4064f586f24b8f1868008e2e623f6c4e74ef381247e49df1
```

The *client.conf*  becomes:
```
user=clientusername
password=92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724
```

The *router.conf* file becomes:
```
user=routerusername
password=2ce01a11893f276a4064f586f24b8f1868008e2e623f6c4e74ef381247e49df1
user_password_dictionary=credentials.conf
```

The *credentials.conf* file becomes:
```
clientusername:92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724
```
