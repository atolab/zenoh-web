---
title: "Pico (embedded-C)"
weight : 6600
menu:
  docs:
    parent: migration_1.0
---

## General API changes

We have reworked the type naming to clarify how they should be interacted with.

### Owned types

Owned types are allocated by the user and it is their responsibility to drop them using `z_drop` (or `z_close` for sessions).

Previously, we were returning Zenoh structures by value. In Zenoh 1.0.0, a reference to memory must be provided. This allows initializing user allocated structures and frees return value for error codes.

Here is a quick example of this change:

- Zenoh 0.11.x

```c
z_owned_session_t session = z_open(z_move(config));
if (!z_check(session)) {
    return -1;
}
z_close(z_move(session));
```

- Zenoh 1.0.0

```c
z_owned_session_t session;
if (z_open(&sesion, z_move(config), &opts) < 0) {
	return -1;
}
z_close(z_move(session))
```

The owned objects have a “null” state.

- If the constructor function (e.g. `z_open`) fails, the owned object will be set to its null state.
- The `z_drop` releases the resources of the owned object and sets it to the null state to avoid double drop. Calling `z_drop` on an object in a null state is safe and does nothing.

(TODO) Owned types support move semantics, which will consume the owned object and turn it into a moved object, see next section.

### Moved types

Moved types are obtained when using `z_move` on an owned type object. They are consumed on use when passed to relevant functions. Any non-constructor function accepting a moved object (i.e. an object passed by an owned pointer) becomes responsible for calling drop on it. The object is guaranteed to be in the null state when the function returns, even if it fails.

### Loaned types

Each owned type now has a corresponding `z_loaned_xxx_t` type, which is obtained by calling 

 `z_loan` or `z_loan_mut` on it, or eventually received from Zenoh functions / callbacks. 

It is no longer possible to directly access the fields of an owned object, the accessor functions on the loaned objects should instead be used.

 Here is a quick example:

- Zenoh 0.11.x

```c
void reply_handler(z_owned_reply_t *reply, void *ctx) {
    if (z_reply_is_ok(reply)) {
        z_sample_t sample = z_reply_ok(reply);
        printf(">> Received ('%.*s')\n", (int)sample.payload.len, sample.payload.start);
    }
}
```

- Zenoh 1.0.0

```c
void reply_handler(const z_loaned_reply_t *reply, void *ctx) {
    if (z_reply_is_ok(reply)) {
        const z_loaned_sample_t *sample = z_reply_ok(reply);
        z_owned_string_t replystr;
        z_bytes_deserialize_into_string(z_sample_payload(sample), &replystr);
        printf(">> Received ('%s')\n", z_string_data(z_loan(replystr)));
    }
}
```

Certain loaned types can be copied into owned, using the `z_clone` function:

```c
void reply_handler(const z_loaned_reply_t *reply, void *ctx) {
    some_struct_t *some_ctx = (some_struct_t *)(ctx);
    z_clone(some_ctx->query, query);
    some_ctx->has_query = true;
}
```

In the case of our callback, this allows the data from the loaned type to be used in another thread context. Note that this other thread should not forget to call `z_drop` to avoid a memory leak!

### View types

View types are only wrappers to user allocated data, like `z_view_keyexpr_t.` These types can be loaned in the same way as owned types but they don’t need to be dropped explicitly (user is fully responsible for deallocation of wrapped data).

- Zenoh 0.11.x

```c
const char *keyexpr = "example/demo/*";
z_owned_subscriber_t sub = z_declare_subscriber(z_loan(session), z_keyexpr(keyexpr), z_move(callback), NULL);
if (!z_check(sub)) {
    printf("Unable to declare subscriber.\n");
    return -1;
}
```

- Zenoh 1.0.0

```c
const char *keyexpr = "example/demo/*";
z_owned_subscriber_t sub;
z_view_keyexpr_t ke;
z_view_keyexpr_from_str(&ke, keyexpr);
if (z_declare_subscriber(&sub, z_loan(session), z_loan(ke), z_move(callback), NULL) < 0) {
    return -1;
}
```

## Payloads and serialization

We changed how payloads are handled. Before you would pass the pointer and the size of your data and now everything must be serialized into `z_owned_bytes_t`. 

We have added a number of serialization functions from primitive types to and from `z_owned_bytes_t` to simplify this.

- Zenoh 0.11.x

```c
char *value = "Some data to publish on Zenoh";

if (z_put(z_loan(session), z_keyexpr(ke), (const uint8_t *)value, strlen(value), NULL) < 0) {
    return -1;
}
```

- Zenoh 1.0.0

```c
char *value = "Some data to publish on Zenoh";
z_owned_bytes_t payload;
z_bytes_serialize_from_str(&payload, value);

if (z_put(z_loan(session), z_loan(ke), z_move(payload), NULL) < 0) {
  return -1;
}
```

## Attachment

Attachments are now a second payload and must also be serialized to `z_owned_bytes_t`. 

We have a serialization function to encode the key/pair value but nothing prevents use to serialize an another type as attachment.

- Zenoh 0.11.x

```c
char *value = "Some data to publish on Zenoh";

z_put_options_t options = z_put_options_default();
z_owned_bytes_map_t map = z_bytes_map_new();
z_bytes_map_insert_by_alias(&map, _z_bytes_wrap((uint8_t *)"test", 2), _z_bytes_wrap((uint8_t *)"attachement", 5));
options.attachment = z_bytes_map_as_attachment(&map);

if (z_put(z_loan(session), z_keyexpr(ke), (const uint8_t *)value, strlen(value), NULL) < 0) {
    return -1;
}

if (z_put(z_loan(s), z_keyexpr(keyexpr), (const uint8_t *)value, strlen(value), &options) < 0) {
    return -1;
}

z_bytes_map_drop(&map);
```

- Zenoh 1.0.0

```c

char *value = "Some data to publish on Zenoh";
z_owned_bytes_t payload;
z_bytes_serialize_from_str(&payload, value);

// It's up to the user to create the item serializer function
z_publisher_put_options_t options;
z_publisher_put_options_default(&options);
z_owned_bytes_t attachment;
z_bytes_serialize_from_iter(&attachment, create_attachment_iter, (void *)&ctx);
options.attachment = z_move(attachment);

if (z_put(z_loan(session), z_loan(ke), z_move(payload), &options) < 0) {
  return -1;
}
```

## Encoding

How encoding is handled has been reworked:  While before you would use an enum id and a value, now you simply need to register your encoding metadata from a string with `z_encoding_from_str`. 

We have a set of constants to do some wire-level optimization. To leverage this, your string needs to follow the format: "<constant>;<optional additional data>"

- Zenoh 0.11.x

```c
char *value = "Some data to publish on Zenoh";
z_put_options_t options = z_put_options_default();
options.encoding = z_encoding(Z_ENCODING_PREFIX_TEXT_PLAIN, "utf8");

if (z_put(z_loan(session), z_keyexpr(ke), (const uint8_t *)value, strlen(value), &options) < 0) {
    return -1;
}
```

- Zenoh 1.0.0

```c
char *value = "Some data to publish on Zenoh";
z_owned_bytes_t payload;
z_bytes_serialize_from_str(&payload, value);

z_publisher_put_options_t options;
z_publisher_put_options_default(&options);
z_encoding_from_str(&encoding, "text/plain;utf8");
options.encoding = z_move(encoding);

if (z_put(z_loan(session), z_loan(ke), z_move(payload), &options) < 0) {
  return -1;
}
```

## Timestamps

We now tie generating a timestamp to a Zenoh session, with the timestamp inheriting the `ZenohID` of the session.

- Zenoh 0.11.x

```c
// Didn't exist in pico
```

- Zenoh 1.0.0

```c
z_timestamp_t ts;
z_timestamp_new(&ts, z_loan(s));
options.timestamp = &ts;
z_publisher_put(z_loan(pub), z_move(payload), &options);
```

## Pull subscribers / channels

Pull subscribers were removed and are replaced by FIFO or ring buffers.

## Interests

We have added interests to the protocol. This allows a few things:

- Replay missed pubs to a late-joining node.
- Activate writer-side filtering: a pub is not sent on the network if there is no corresponding sub.