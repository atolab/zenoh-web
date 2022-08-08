---
title: "Abstractions"
weight: 2010
menu:
  docs:
    parent: manual
---

Zenoh is a **distributed service** to define, manage and operate on **key/value** spaces.

The key abstractions at the core of zenoh are the following:

## Key

Zenoh operates on **key/value** pairs. The most important thing to know about Zenoh keys is that `/` is the hierarchical separator, just like in unix filesystems. While you could set up your own hierarchy using other separators, your zenoh exchanges would likely suffer much worse performance, as using `/` will let Zenoh do clever optimisations (users have informed us in the past that switching from `.` to `/` as their hierarchy-separator almost divided their CPU usage by 2).

However, you will much more often interact with [key expressions](#key-expressions), which provide a small regular language to match sets of keys.

There are a few restrictions on what may be a key:
- A character in a key should be a non-empty UTF-8 string.
- An individual key may not contain the characters `*`, `$`, `?`, `#`.
- ~~Some characters such as `?`, `(`, `)`, `[` and `]` are not explicitly forbidden, but may interact very poorly with the [selector](#selector) syntax. We may forbid them from existing in key-expression in future releases.~~ ~~Using `?`, `(`, `)`, `[` and `]` are discouraged since it will interfere with the [selector](#selector) syntax. We may forbid them in future releases.~~
- ~~While it likely won't affect Zenoh's inner-workings, we advise that your keys remain UTF-8 strings, as most tools that display keys will assume that they are and suffer render-issues if not.~~ 

A typical Zenoh key would look something like `organizationA/building8/room275/sensor3/temperature`.

---

## Key Expressions

Key expressions allow you to adress a set of keys in a single request. This can be used for convenience, or as a bandwidth-saving measure to avoid making the same request multiple times with only small changes in the key.

Key expressions are a small regular language, where `*` behaves much like it does in globs:
- `*` matches any set of characters in a key, except `'/'`. It can only be surrounded by `/`.
  For example, subscribing to `organizationA/building8/room275/*/temperature` will ensure that any temperature message from any device in room 275 of building 8 will be routed to your subscriber.  
  Note however that this expression wouldn't match `organizationA/building8/room275/temperature`.
- `$*` is like `*` except it may be surrounded by any other characters.
  For example, subscribing to `organizationA/building8/room275/thermometer$*/temperature` will get the temperature readings from all thermometers in the room.
- `**` is equivalent to `.*` in regular expression syntax: it will match absolutely anything, including nothing. They may appear at the beginning of a key expression or after a `/`, and `/` is the only allowed character after a `**`  
  For example, subscribing to `organizationA/**/temperature` will ensure that any temperature message from ALL devices in organization A.

The [Key Expression Language](https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Key%20Expressions.md) is defined [here](https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Key%20Expressions.md).

### Optimizing key expressions

While the language is simple, it hides sometimes complex algorithms and behaviours. Forming your key expressions well can be the key to reducing the ressource-requirements of your Zenoh infrastructure:
- Make your expressions as precise as possible: `organizationA/building8/room275/*/temperature` and `organizationA/building8/room275/thermometer$*/temperature` are similar, but if your rooms also contain `robot12` and `pc18` (even though neither exposes a direct `temperature` child), specifying that you're only interested in thermometers will reduce matching costs on the infrastructure.
- Avoid middle-`**`: while trailing-`**` are very low-cost, middle-`**` such as the one in the previous example are generally more costly to evaluate. If your hierarchy is well-formed, you should be able to replace a middle-`**` by an appropriately long chain of `*` segments, such as `organizationA/*/*/*/temperature` for our example, which will be much cheaper to evaluate.
- You may sometimes want to use a combination of `*` and `**` to express that you want to match "any key that is at least this deep". For example `**/*/*/temperature` would match any key that contains at least 3 segments with the last one matching `temperature`. When doing so, bubble up your `*`s to the left, to obtain the canonical form `*/*/**/temperature`. By doing so, you will significantly reduce the cost of evaluating matches for that expression.

---

## Selector

A selector is an extension of the [key expression](#key-expressions) syntax, and is made of two parts: 
- The key expression, which is the part of the selector that routers will consider when routing a Zenoh message.
- Optionally, separated from the key expression by a `?`, the value-selector acts as arguments for a Zenoh query.

Queryables are free to interpret the value-selector however they see fit, but Zenoh-provided [queryables](#queryable-previously-named-eval), such as the [admin-space](#admin-space) will generally use the the same format as a URL query section, that we encourage queryable-implementers to use also for consistency.

A value-selector is abstracted as a list of key-value pairs, formatted such that:
- key-value pairs are separated by &.
- each pair follows the (?<key>[^=]*)(=(?<value>[^&]*))? regex: note that the value section is optional: somekey parses to ("somekey", "").
- both the key and value fields shall be URL-encoded to allow for character escapement.

<!-- A zenoh-typical value selector is made of 3 sections, in that order:
- The **filter** section is a `&` separated list of predicates, such that the queryable should filter out values that don't fulfill all predicates.  
  Each predicate has the form `<field><operator><literal>`. `<field>` is the name of a field in the value. If that field is not found or has an unexpected type, the predicate is considered unfulfilled. The `<operator>` may be `<`, `>`, `<=`, `>=`, `=` or `!=`. The `<literal>` is the value for the comparison.
  _For now, very few queryables actually support this element_
- The **arguments** (previously named **properties**) section, surrounded by parentheses, serves as a list of named arguments for the queryable. For example, time-series queryable have a `starttime` argument used to mark that no value older than that time is wanted by the querier.
- The **projection** (or **fragment**) section, surrounded by square brackets, is a list of field-names, such that only these fields will be returned for each value. -->

```none
/s1/s2/.../sn?p1=v1&p2=v2&...&_filter=x>1ANDy<2AND...ORz=4&_projection=a;b;x;y;...;z
|           | |             |          |             |                |             |
|           | |- arguments--|          |--- filter --|                |--projection-|
|- key expr | |-------------------- value selector ---------------------------------|
```

Any key that doesn't start with an alphanumeric character is reserved for Zenoh internals. Zenoh may adapt its behavior to the presence and values associated with any key that doesn't start with an alphanumeric character.
Zenoh supports the following keys currently:
- `_time` property is reserved queryables that maintain a history of values are encouraged to use the _time key
as a way to obtain the range of time the querier is interested in.
- `_filter` property is reserved to specify filtering function to be applied on the query result.
- `_projection` property is reserved for the querier to specify the projection of values.
To avoid collisions of keys between queryables that may treat them with different semantics, implementers are encouraged to select a prefix for their own keys.


---

## Value

A user provided data item along with its [encoding](#encoding).

---

## Encoding

A description of the [value](#value) format, allowing zenoh (or your application) to know how to encode/decode the value to/from a bytes buffer.

By default, zenoh is able to transport and store any format of data as long as it's serializable as a bytes buffer.
But for advanced features such as content filtering (using [selector](#selector)) or to automatically deserialize the data into a concrete type in the client APIs, zenoh requires a description of the data encoding.

Some noteworthy supported encodings are:
- **TextPlain**: the value is a UTF-8 string
- **AppJson** or **TextJson**: the value is a JSON string
- **AppProperties**: the value is a string representing a list of keys/values separated by `';'` (e.g. `"k1=v1;k2=v2..."`), where both key and value are string-typed.
- **AppInteger**: the value is an integer
- **AppFloat**: the value is a float

You may refer to [Zenoh's Rust API documentation](https://docs.rs/zenoh) to get more information on the supported encodings.

You may also write your own encodings by either suffixing an existing one, or by suffixing the `EMPTY` encoding, if you wish to use encodings that are unknown to Zenoh. While Zenoh will not be able to deserialize these encodings, it will expose them to your application so that it may be informed on how it should deserialize any received value.

---

## Timestamp

When a [value](#value) is put into zenoh, the first zenoh router receiving this value automatically
associates it with a timestamp.  
This timestamp is made of 2 items:

- A **time** generated by a [Hybrid Logical Clock (HLC)](https://cse.buffalo.edu/tech-reports/2014-04.pdf).
  This time is a 64-bit time with a similar structure than a NTP timestamp (but with a different epoch):
  - The higher 32-bit part is the number of seconds since midnight, January 1, 1970 UTC
    (implying a rollover in 2106).
  - The lower 32-bit part is a fraction of second, but with the 8 last bits replaced by a counter.

  This time gives a theoritical resolution of 2^-32 seconds (60 nanoseconds), and
  guarantees that the same time cannot be generated twice and that the _happened-before_ relationship is preserved.

- The **UUID** of the zenoh router that generated the time.

Such a timestamp allows zenoh to guarantee that each value introduced into the system has a unique timestamp, and that those timestamps (and therefore the values) can be ordered in the same way at any point of the system, without the need of any consensus algorithm.

---

## Subscriber

An entity registering interest for being notified whenever a key/value with a key matchings the subscriber
[selector](#selector) is put, updated or removed on zenoh.

---

## Queryable (previously named Eval)

A computation registered at a specific [key expression](#key-expressions).

This computation can be triggered by a `get` operation on a [selector](#selector) matching this key expression.
The computation function will receive the selector's properties as parameters.

---

## Storage

[Storages](../plugin-storage-manager) are an intersection of a queryable and subscriber:
- They subscribe to a set of keys;
- upon receiving publications onto a subset of their subscription set, they store the associated values;
- when queried about a subset of their subscription set, they return the latest values for each appropriate key.

`zenohd`, the reference implementation of a Zenoh node, supports storages through the [`storages` plugin](../plugin-storage-manager).

Since there exist many ways to implement the storing part of the process, the `storages` plugin relies on dynamically loaded [volumes](../plugin-storage-manager#backends-and-volumes) to do the actual value-storing. Each volume has its own tradeoffs, as well as potential uses besides acting as a database for `zenohd`.

---

## Admin space

The administration space of zenoh allowing to administrate a zenoh router and its plugins.
It is accessible via regular get/put on zenoh, under the `@/router/<router-id>` prefix, where
**`<router-id>`** is the UUID of a zenoh router.

When using the REST API, you can replace the `<router-id>` with the **`local`** keyword,
meaning the operation addresses the zenoh router the HTTP client is connected to.

For instance, the following keys can be used:

- `@/router/<router-id>` (read-only):  
  Returns a JSON with the status information about the router.
- `@/router/<router-id>/config/**` (write-only):  
  Allows you to edit the configuration of the router at runtime.

Some plugins may extend the admin space, such as [Storages](../plugin-storage-manager), which will add the following keys:

- `@/router/<router-id>/status/plugins/storage_manager/volumes/<volume-name>` (read-only):  
  Returning information about the selected backend in JSON format
- `@/router/<router-id>/status/plugins/storage_manager/storages/<storage-name>` (read-only):  
  Returning information about the selected storage in JSON format
