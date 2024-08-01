---
title: "Access Control"
weight : 3900
menu:
  docs:
    parent: manual
---

*Access control* enables Zenoh instances to filter (allow or deny) messages, depending on certain characteristics of individual messages and their respective source or destination. *Authentication* on the other hand allows Zenoh instances to identify certain characteristics in other instances they connect to, which are used to match the instances with their roles in the ACL policies and apply the rules accordingly on the exhanged messages.

The configuration of access control policies is done via a [configuration file](../configuration).

---------
## ACL configuration

ACL configuration mainly consists of three components: `rules`, `subjects` and `policies`, to which is added the `default_permission` (`allow` or `deny`) to be applied on messages that do not match the configured policies.

The `enabled` boolean field allows to enable and disable ACL. Note that ACL config cannot be updated at runtime, and requires a restart of the instance to reflect the changes.

### rules

Each rule within the `rules` list is identified by a unique `id` string. Rules apply on matched messages based on their individual characteristics:

- `messages`: types of messages to apply the rule on. Supported message types are the following:
  - Declare Subscriber (`declare_subscriber`)
  - Publication (`put`)
  - Delete (`delete`)
  - Declare Queryable (`declare_queryable`)
  - Query (`query`)
  - Reply (`reply`)
- `flows`: applies rule on incoming messages (`ingress`), outgoing messages (`egress`), or both directions. If this field is not provided in the config, the rule will apply to both directions by default.
- `key_exprs`: the rule applies on messages for which the key expression matchs one of the given key expressions. For more details on key expression matching, see [Key Expressions](http://localhost:1313/docs/manual/abstractions/#key-expression).

Matched messages are filtered based on the rule's `permission`: `allow` or `deny`.

For instance, the following rule denies all incoming and outgoing subscriptions, publications and deletions on key expressions matching `demo/example/**`:

```json
{
  "id": "deny pub/sub",
  "permission": "deny",
  "flows": ["ingress", "egress"],
  "messages": [
    "declare_subscriber",
    "put",
    "delete",
  ],
  "key_exprs": [
    "demo/example/**",
  ],
}
```

### subjects

Each subject configuration is identified by a unique `id` string. Subject configuration combines `interfaces`, `cert_common_names` and `usernames` to match with configurations of connecting zenoh instances.

- `interfaces`: list of local network interfaces through which the configured instance communicates with the remote instance to be matched.
- `cert_common_names`: list of certificate common names which are matched with the respective certificate content of the remote instance using TLS or Quic transport. For details on certificate configuration refer to TLS [TLS authentication](../tls) and [QUIC transport](../quic).
- `usernames`: list of usernames to be matched with the authentication config of remote instances. Refer to [User-Password authentication](../user-password) for how to setup this authentication mechanism.

To produce all possible combinations that characterize a subject configuration, the cartesian product of the `interfaces`, `cert_common_names` and `usernames` lists is calculated. Below is an example of a subject and its internal representation.

```json
{
  "id": "example subject",
  "interfaces": [
    "lo0",
    "en0",
  ],
  "cert_common_names": [
    "example.zenoh.io"
  ],
  "usernames": [
    "zenoh-example1",
    "zenoh-example2",
  ],
  // This instance translates internally to this filter:
  // (interface="lo0" && cert_common_name="example.zenoh.io" && username="zenoh-example1") ||
  // (interface="lo0" && cert_common_name="example.zenoh.io" && username="zenoh-example2") ||
  // (interface="en0" && cert_common_name="example.zenoh.io" && username="zenoh-example1") ||
  // (interface="en0" && cert_common_name="example.zenoh.io" && username="zenoh-example2")
}
```

Note that any of the three lists presented above can be ommited, and will be interpreted as a wildcard (i.e matches with all possible values of that authentication method). This implies that the empty subject below is a wildcard that will match any zenoh instance.

```json
{
  "id": "subject that matches all zenoh instances",
}
```

### policies

The `policies` list associates configured rules to configured subjects based on their unique `id`s. For example, the config below creates the `deny pub/sub` rule and the `example subject` subject then associates them in a policy.

```json
{
  access_control: {
    "enabled": true,
    "default_permission": "allow",
    "rules": [
      {
        "id": "deny pub/sub",
        "permission": "deny",
        "flows": ["ingress", "egress"],
        "messages": [
          "declare_subscriber",
          "put",
          "delete",
        ],
        "key_exprs": [
          "demo/example/**",
        ],
      }
    ],
    "subjects": [
      {
        "id": "example subject",
        "interfaces": [
          "lo0",
          "en0",
        ],
        "cert_common_names": [
          "example.zenoh.io"
        ],
        "usernames": [
          "zenoh-example1",
          "zenoh-example2",
        ],
      }
    ],
    "policies": [
      {
        "rules": [
          "deny pub/sub"
        ],
        "subjects": [
          "example subject"
        ],
      },
    ]
  }
}
```

---------

For a more technical analysis of the ACL feature, please refer to the [Access Control Rules RFC](https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Access%20Control%20Rules.md).