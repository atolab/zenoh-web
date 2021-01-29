---
title: "User-Password authentication"
weight : 2200
menu:
  docs:
    parent: manual
---

Zenoh supports basic *user-password authentication*.
Clients and peers can use *user* and *password* for authentication against a router or a peer.
The configuration of credentials is done via a configuration file defining certain [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html).

---------
## Client configuration

The required [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for basic *user-password authentication* for a client are **user** and **password**.
A configuration file for a *client* would be:
```
user=clientusername
password=clientpassword
```

When using such configuration, the client will use the provided **user** and **password** to authenticate against any peer or router.

Let's assume that the above configuration is then saved with the name *client.conf*.

## Router or peer configuration

The required [zenoh properties](https://docs.rs/zenoh/0.5.0-beta.5/zenoh/net/config/index.html) for basic *user-password authentication* for a client are **user**, **password**, **user_password_dictionary**.
The **user_password_dictionary** zenoh property indicates the path of a text file containing the full list of authorized credentials for connections.

A configuration file for a *router* or *peer* would be:
```
user=routerusername
password=routerpassword
user_password_dictionary=credentials.conf
```

The authorized credentials text file would hence be:
```
clientusername:clientpassword
```
Each line of the file represents the one signle authorized credential in the form of *user:password*.
The authorized credentials text file can contain any number of lines.

When using such configuration, the router or peer will use the provided **user** and **password** to authenticate against any other peer or router.
At the same time, it will use the file containing the authorized credentials to authenticate incoming conncetions.

Let's assume that the above configurations are then saved with the name *router.conf* and *credentials.conf*.


---------
## Testing the user-password authentication

Let's assume a scenario with one zenoh router and two clients connected to it: one publisher and one subscriber.

The first thing to do is to run the router passing its configuration, i.e. *router.conf*:
```bash
$ zenohd -c router.conf
```
Make sure that the path indicated in the *user_password_dictionary* property points to a valid credentials file.

Then, let's start the subscriber in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_sub --mode='client' -c client.conf
```

Lastly, let's start the publisher in client mode passing its configuration, i.e. *client.conf*:
```bash
$ zn_pub --mode='client' -c client.conf
```

As it can be noticed, the same *client.conf* is used for *z_sub* and *z_put*. 
Both are using the same credentials and are authenticated accordingly by the *router*. 
Neverhteless, different configuration files and credentials could be used 

---------
## Final remark

Consider to not store clear text password in the configuration files. Before creating the configuration files, an hashing function should be used to hash the password. 
For example, let's compute the hash for the *client* and *router* passwords:
```bash
$ echo clientpassword | sha256sum
92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724

$ echo routerpassword | sha256sum
2ce01a11893f276a4064f586f24b8f1868008e2e623f6c4e74ef381247e49df1
```

Therefore, the *client.conf* would become:
```
user=clientusername
password=92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724
```

And the *router.conf* file would become:
```
user=routerusername
password=2ce01a11893f276a4064f586f24b8f1868008e2e623f6c4e74ef381247e49df1
user_password_dictionary=credentials.conf
```

Finally, the *credentials.conf* file would become:
```
clientusername:92df0d4f39c218607e3200bf93ccac87a80cb910e811d84b286bebe0a8860724
```
