---
title: "C / Pico"
weight : 6400
menu:
  docs:
    parent: migration_1.0
---

## General API changes

We have reworked the type naming to clarify how types should be interacted with.

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

Moved types are obtained when using `z_move` on an owned type object. They are consumed on use when passed to relevant functions. Any non-constructor function accepting a moved object (i.e. an object passed by owned pointer) becomes responsible for calling drop on it. The object is guaranteed to be in the null state upon such function return, even if it fails.

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

## Payload and Serialization

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

To implement custom (de-)serialization functions Zenoh 1.0.0 provides `z_bytes_iterator_t`, `z_bytes_reader_t` and `z_owned_bytes_wrtiter_t` types and corresponding functions. Alternatively it is always possible to perform serialization separately and only send/receive `uint8_t` arrays, by only calling trivial `z_bytes_serialize_from_slice` and `z_bytes_deserialize_into_slice` functions:

```cpp
void send_data(const z_loaned_publisher_t* pub, const uint8_t *data, size_t len) {
	z_owned_bytes_t payload;
  z_bytes_serialize_from_slice(&payload, data, len);
  z_publisher_put(pub, z_move(payload), NULL);
  // no need to drop the payload, since it is consumed by z_publisher_put
}	

void receive_data(const z_loaned_bytes_t* payload) {
	z_owned_slice_t slice;
	z_bytes_deserialize_into_slice(payload, &slice);
	
	// do something with serialized data
	// raw ptr can be accessed via z_slice_data(z_loan(slice))
	// data length can be accessed via z_slice_len(z_loan(slice))
	
	// in the end slice should be dropped since it contains a copy of the payload data
	z_drop(z_move(slice)); 
}
```

Note that it is no longer possible to access the underlying payload data pointer directly, since Zenoh cannot guarantee that the data is delivered as a single fragment. So in order to get access to raw payload data one must use `z_bytes_reader_t` and related functions:

```cpp
z_bytes_reader_t reader = z_bytes_get_reader(z_loan(payload));
uint8_t data1[10] = {0};
uint8_t data2[20] = {0};

z_bytes_reader_read(&reader, data1, 10); // copy first 10 payload bytes to data1
z_bytes_reader_read(&reader, data2, 20); // copy next 20 payload bytes to data2
```

## Stream Handlers and Callbacks

Prior to 1.0.0 stream handlers were only supported for `z_get`and `z_owned_queryable_t`:

```cpp
// callback
z_owned_closure_reply_t callback = z_closure(reply_handler);
z_get(z_loan(session), z_keyexpr(keyexpr), "", z_move(callback), &opts);

// stream handlers interface
// blocking
z_owned_reply_channel_t channel = zc_reply_fifo_new(16);
z_get(z_loan(session), z_keyexpr(keyexpr), "", z_move(channel.send), &opts);
z_owned_reply_t reply = z_reply_null();
for (z_call(channel.recv, &reply); z_check(reply); z_call(channel.recv, &reply)) {
    if (z_reply_is_ok(&reply)) {
	    z_sample_t sample = z_reply_ok(&reply);
	    // do something with sample and keystr
		} else {
			printf("Received an error\n");
		}
    z_drop(z_move(reply));
}
z_drop(z_move(channel));

// non-blocking
z_owned_reply_channel_t channel = zc_reply_non_blocking_fifo_new(16);
z_get(z_loan(s), keyexpr, "", z_move(channel.send), &opts);
z_owned_reply_t reply = z_reply_null();
for (bool call_success = z_call(channel.recv, &reply); !call_success || z_check(reply);
    call_success = z_call(channel.recv, &reply)) {
    if (!call_success) {
        continue;
    }
    if (z_reply_is_ok(z_loan(reply))) {
	    z_sample_t sample = z_reply_ok(&reply);	    
			// do something with sample
		} else {
			printf("Received an error\n");
		}
		z_drop(z_move(reply));
}
z_drop(z_move(channel));
```

In 1.0.0 `z_owned_subscriber_t`, `z_owned_queryable_t` and `z_get` can use either a callable object or a stream handler. In addition the same handler type now provides both blocking and non-blocking interface. For the time being Zenoh provides 2 types of handlers: 

- `FifoHandler` - serving messages in Fifo order, when it is full, receiving will be blocked until messages in the queue are consumed and space is freed up.
- `RingHandler` - serving messages in Fifo order, will remove older messages to make room for new ones when the buffer is full.

```cpp
// callback
z_owned_closure_reply_t callback = z_closure(reply_handler);
z_get(z_loan(session), z_keyexpr(keyexpr), "", z_move(callback), &opts);

// stream handlers interface
z_owned_fifo_handler_reply_t handler;
z_owned_closure_reply_t closure;
z_fifo_channel_reply_new(&closure, &handler, 16);
z_get(z_loan(s), z_loan(keyexpr), "", z_move(closure), z_move(opts));
z_owned_reply_t reply;

// blocking
for (bool is_alive = z_recv(z_loan(handler), &reply); is_alive; is_alive = z_recv(z_loan(handler), &reply)) {
    // z_recv will block until there is at least one sample in the Fifo buffer
    if (z_reply_is_ok(z_loan(reply))) {
        const z_loaned_sample_t *sample = z_reply_ok(z_loan(reply));
        // do something with sample
    } else {
        printf("Received an error\n");
    }
    z_drop(z_move(reply));
}

// non-blocking
for (bool is_alive = z_try_recv(z_loan(handler), &reply); is_alive; is_alive = z_try_recv(z_loan(handler), &reply)) {
    if (!z_check(reply)) {
	    continue; // z_try_recv is non-blocking call, so will fail to return a reply if the Fifo buffer is empty
    }
    if (z_reply_is_ok(z_loan(reply))) {
        const z_loaned_sample_t *sample = z_reply_ok(z_loan(reply));
        // do something with sample
    } else {
        printf("Received an error\n");
    }
    z_drop(z_move(reply));
}
  
```

Same works for `Subscriber` and `Queryable` :

```cpp
// callback
void data_handler(const z_loaned_sample_t *sample, void *context) {
	// do something with sample
}

z_owned_closure_sample_t callback;
z_closure(&callback, data_handler, NULL, NULL);
z_owned_subscriber_t sub;
if (z_declare_subscriber(&sub, z_loan(session), z_loan(keyexpr), z_move(callback), NULL) < 0) {
  printf("Unable to declare subscriber.\n");
  exit(-1);
}

// stream handlers interface
z_owned_fifo_handler_sample_t handler;
z_owned_closure_sample_t closure;
z_fifo_channel_reply_new(&closure, &handler, 16);
z_owned_subscriber_t sub;
if (z_declare_subscriber(&sub, z_loan(session), z_loan(keyexpr), z_move(closure), NULL) < 0) {
  printf("Unable to declare subscriber.\n");
  exit(-1);
}

z_owned_sample_t sample;
// blocking
for (bool is_alive = z_recv(z_loan(handler), &sample); is_alive; is_alive = z_recv(z_loan(handler), &sample)) {
    // z_recv will block until there is at least one sample in the Fifo buffer
		// it will return an empty sample and is_alive=false once subscriber gets disconnected
    
    // do something with sample
    z_drop(z_move(sample));
}

// non-blocking
for (bool is_alive = z_try_recv(z_loan(handler), &sample); is_alive; is_alive = z_try_recv(z_loan(handler), &sample)) {
    if (!z_check(sample)) { // z_try_recv is non-blocking call, so will fail to return a sample if the Fifo buffer is empty
	    continue;
    }
		// do something with sample
    z_drop(z_move(sample));
}
```

The `z_owned_pull_subscriber_t` was removed, given that `RingHandler` can provide similar functionality with ordinary `z_owned_subscriber_t.`

## Attachment

Prior to 1.0.0 it attachment could only represent a set of key-value pairs and was represented by a separate type:

```c
// publish message with attachment
char *value = "Some data to publish on Zenoh";

z_put_options_t options = z_put_options_default();
z_owned_bytes_map_t map = z_bytes_map_new();
z_bytes_map_insert_by_alias(&map, _z_bytes_wrap((uint8_t *)"test", 2), _z_bytes_wrap((uint8_t *)"attachement", 5));
options.attachment = z_bytes_map_as_attachment(&map);

if (z_put(z_loan(s), z_keyexpr(keyexpr), (const uint8_t *)value, strlen(value), &options) < 0) {
    return -1;
}

z_bytes_map_drop(&map);

// receive sample with attachment

int8_t attachment_reader(z_bytes_t key, z_bytes_t val, void* ctx) {
    printf("   attachment: %.*s: '%.*s'\n", (int)key.len, key.start, (int)val.len, val.start);
    return 0;
}

void data_handler(const z_sample_t* sample, void* arg) {
    // checks if attachment exists
    if (z_check(sample->attachment)) {
        // reads full attachment
        z_attachment_iterate(sample->attachment, attachment_reader, NULL);

        // reads particular attachment item
        z_bytes_t index = z_attachment_get(sample->attachment, z_bytes_from_str("index"));
        if (z_check(index)) {
            printf("   message number: %.*s\n", (int)index.len, index.start);
        }
    }
    z_drop(z_move(keystr));
}

```

In 1.0.0 attachment handling was greatly simplified. It is now represented as `z_..._bytes_t` (i.e. the same type we use to represent serialized data). So it can now contain data in any format (and not only be restricted to key-value pairs). 

```c
// publish attachment
typedef struct {
	char* key;
	char* value;
} kv_pair_t;

typedef struct kv_it {
    kv_pair_t *current;
    kv_pair_t *end;
} kv_it;

bool create_attachment_iter(z_owned_bytes_t* kv_pair, void* context) {
  kv_it* it = (kv_it*)(context);
  if (it->current == it->end) { 
    return false;
  }
  z_owned_bytes_t k, v;
  // serialize as key-value pair
  z_bytes_serialize_from_str(&k, it->current->key);
  z_bytes_serialize_from_str(&v, it->current->value);
  z_bytes_serialize_from_pair(kv_pair, z_move(k), z_move(v));
  it->current++;
  return true;
};

...

kv_pair_t attachment_kvs[2] = {;
	(kv_pair_t){.key = "index", .value = "1"},
	(kv_pair_t){.key = "source", .value = "C"}
}
kv_it it = { .begin = attachment_kvs, .end = attachment_kvs + 2 };

z_owned_bytes_t payload, attachment;
// serialzie key value pairs as attachment using z_bytes_serialize_from_iter
z_bytes_serialize_from_iter(&attachment, create_attachment_iter, (void*)&it);
options.attachment = &attachment;

z_bytes_serialize_from_str(&payload, "payload");
z_publisher_put(z_loan(pub), z_move(payload), &options);

// receive sample with attachment

void data_handler(const z_loaned_sample_t *sample, void *arg) {
  z_view_string_t key_string;
  z_keyexpr_as_view_string(z_sample_keyexpr(sample), &key_string);

  z_owned_string_t payload_string;
  z_bytes_deserialize_into_string(z_sample_payload(sample), &payload_string);

	printf(">> [Subscriber] Received %s ('%.*s': '%.*s')\n", kind_to_str(z_sample_kind(sample)),
   (int)z_string_len(z_loan(key_string)), z_string_data(z_loan(key_string)),
   (int)z_string_len(z_loan(payload_string)), z_string_data(z_loan(payload_string)));
	z_drop(z_move(payload_string));
	
  const z_loaned_bytes_t *attachment = z_sample_attachment(sample);
  // checks if attachment exists
  if (attachment == NULL) {
    return;
	}
	
  // read attachment key-value pairs using bytes_iterator
  z_bytes_iterator_t iter = z_bytes_get_iterator(attachment);
  z_owned_bytes_t kv;
  while (z_bytes_iterator_next(&iter, &kv)) {
      z_owned_bytes_t k, v;
      z_owned_string_t key, value;
      z_bytes_deserialize_into_pair(z_loan(kv), &k, &v);

      z_bytes_deserialize_into_string(z_loan(k), &key);
      z_bytes_deserialize_into_string(z_loan(v), &value);

      printf("   attachment: %.*s: '%.*s'\n",
        (int)z_string_len(z_loan(key)), z_string_data(z_loan(key)),
        (int)z_string_len(z_loan(value)), z_string_data(z_loan(value)));
      z_drop(z_move(kv));
      z_drop(z_move(k));
      z_drop(z_move(v));
      z_drop(z_move(key));
      z_drop(z_move(value));
  }
}
```

## Encoding

Encoding handling has been reworked:  before one would use an enum id and a string suffix value, now it is needed only to register the encoding metadata from a string with `z_encoding_from_str`. 

There is a set of predefined constant encodings subject to some wire-level optimization. To benefit from this, the your string should follow the format: "<predefined constant>;<optional additional data>"

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

We now introduce timestamp generation tied to a Zenoh session, with the timestamp inheriting the `ZenohID` of the session.

- Zenoh 0.11.x

```c
// Didn't exist
```

- Zenoh 1.0.0

```c
z_timestamp_t ts;
z_timestamp_new(&ts, z_loan(s));
options.timestamp = &ts;
z_publisher_put(z_loan(pub), z_move(payload), &options);
```