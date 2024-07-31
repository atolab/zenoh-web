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

ACL configuration mainly consists of three components: `rules`, `subjects` and `policies`.

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

```
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
