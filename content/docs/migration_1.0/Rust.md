---
title: "Rust"
weight : 6200
menu:
  docs:
    parent: migration_1.0
---


## Module reorganization

We reorganized the module tree, so import paths are not the same as before. The main difference is that everything should be imported via the root path `zenoh::`. Here are some examples, but you can look into `zenoh/src/lib.rs` for the complete list of changes.

```rust
// common use
use zenoh::config::*;
use zenoh::{Config, Error, Result};

// key_expr & selector
use zenoh::key_expr::{
    format::{kedefine, keformat},
    keyexpr, KeyExpr, OwnedKeyExpr,
};

// session
use zenoh::session::{init, open, EntityId, Session, SessionInfo};

// publisher & subscriber
use zenoh::pubsub::{Publisher, Reliability, Subscriber};

// query & queryable & selectors
use zenoh::query::{
    ConsolidationMode, Parameters, Query, QueryConsolidation, QueryTarget, Queryable, Reply,
    Selector,
};

// ZBytes & encoding
use zenoh::bytes::{ZBytes, Encoding};

// sample
use zenoh::sample::{Locality, Sample};

// quality of service
use zenoh::qos::{CongestionControl, Priority, QoSBuilderTrait};
```

## Removal of the sync and async preludes

Zenoh preludes has been deprecated and are no more used in the API. The API has also been made asynchronous first: all operations like put/get/etc. can be awaited directly.
Making synchronous calls now requires to import `zenoh::Wait`, and use `wait()` method, replacing the old `res()` method.
To make the migration easier, there is a deprecation prompt if you use the old API convention.

```rust
// (deprecated) async
use zenoh::prelude::r#async::*;
let session = zenoh::open(config).res().await.unwrap();
let publisher = session.declare_publisher(&key_expr).res().await.unwrap();
put.res().await.unwrap();
// (deprecated) sync
use zenoh::prelude::sync::*;
let session = zenoh::open(config).res().unwrap();
let publisher = session.declare_publisher(&key_expr).res().unwrap();
put.res().unwrap();
```

- Zenoh 1.0.0

```rust
// async
// Difference 1: No more res()
let session = zenoh::open(config).await.unwrap();
let publisher = session.declare_publisher(&key_expr).await.unwrap();
publisher.put(buf).await.unwrap();
// sync
// Difference 2: use wait() for synchronous API
use zenoh::Wait;
let session = zenoh::open(config).wait().unwrap();
let publisher = session.declare_publisher(&key_expr).wait().unwrap();
publisher.put(buf).wait().unwrap();
```

## `Session` is now clonable and can be closed easily

`Session` implements `Clone` now, so there is no more need to wrap it into an `Arc<Session>`, and `Session::into_arc` has been deprecated. All the session methods, except `Session::close`, works like before, so only the session type need to be changed.
<br>
As a side effect, `Subscriber` and `Queryable` no longer have a generic lifetime parameter. `Publisher` also looses one of its lifetime parameters, to keep only the one of its key expression.

The session is now closed automatically when the last `Session` instance is dropped, **even if publishers/subscribers/etc. are still alive**. Session can also be manually closed using `Session::close`, which now takes an immutable reference, so it can be called anytime, even if publishers/subscribers/etc. are still alive.
<br>
Subscriber and queryable of a closed session will no longer receive data; trying to call `Session::get`, `Session::put` or `Publisher::put` will result in an error. Closing session on the fly may save bandwidth on the wire, as it avoids propagating the undeclaration of remaining entities like subscribers/queryables/etc.  

```rust
let session = zenoh::open(zenoh::config::peer()).await.unwrap();
let subscriber = session
    .declare_subscriber("key/expression")
    .await
    .unwrap();
let subscriber_task = tokio::spawn(async move {
    while let Ok(sample) = subscriber.recv_async().await {
        println!("Received: {} {:?}", sample.key_expr(), sample.payload());
    }
});
// session can be closed while subscriber is still running, preventing it
// receiving more data
session.close().await.unwrap();
// subscriber task will end as `subscriber.recv_async()` will return `Err`
// **when all remaining data has been processed**.
// subscriber undeclaration has not been sent on the wire
subscriber_task.await.unwrap()
```

## Callbacks run in background until session is closed

Session entities, e.g. subscribers, declared with callbacks are no longer undeclared when they are dropped; there is no longer need to keep a reference to an entity when the intent is to have it run until the session is closed.

```rust
let session = zenoh::open(zenoh::config::default()).await.unwrap();
session
    .declare_subscriber("key/ expression")
    .callback(|sample| { println!("Received: {} {:?}", sample. key_expr(), sample. payload()) })
    .await
    .unwrap();
// subscriber run in background until the session is closed
// no need to keep a variable around
```

If you still want the entity to be undeclared when dropped, you can simply use `with` instead of `callback`; it may just require you to annotate the callback, as type inference is not as good as with `callback` method.

```rust
let session = zenoh::open(zenoh::config::default()).await.unwrap();
let subscriber = session
    .declare_subscriber("key/ expression")
    // annotation needed
    .with(|sample: Sample| { println!("Received: {} {:?}", sample. key_expr(), sample. payload()) })
    .await
    .unwrap();
// subscriber is undeclared when dropped
```

*Going into details, a new method `undeclare_on_drop(bool)` – default to `true`, has been added to the builders, and `callback(cb)` is now simply a shortcut to `with(cb).undeclare_on_drop(false)`. 
However, the normal user would rarely need to call this method directly.*

## Value is gone, long live ZBytes

`Value` has been split into `ZBytes` and `Encoding`. `put` and other operations now require a `ZBytes` payload, and builders accept an optional `Encoding` parameter. The encoding is no longer automatically deduced from the payload type.

`ZBytes` is a raw bytes container, which can also contain non-contiguous regions of memory. It can be created directly from raw bytes/strings using `ZBytes::from`. The bytes can be retrieved using `ZBytes::to_bytes`, which returns a `Cow<[u8]>`, as a copy may have to be done if the underlying bytes are not contiguous.

- Zenoh 0.11.x

```rust
let sample = subscriber.recv_async().await.unwrap();
let value: Value = sample.value;
let raw_bytes: Vec<u8> = value.try_into().unwrap();
```

- Zenoh 1.0.0

```rust
let sample = subscriber.recv_async().await.unwrap();
let zbytes: ZBytes = sample.payload();
let raw_bytes: Cow<[u8]> = zbytes.as_bytes();
```

You can look at a full set of examples in `examples/examples/z_bytes.rs`.

## Serialization

Zenoh does provide serialization for convenience as an extension in the `zenoh-ext` crate. Serialization is implemented for a bunch of standard types like integers, floats, `Vec`, `HashMap`, etc. and is used through functions `z_serialize`/`z_deserialize`.

```rust
let input: Vec<f32> = vec![0.0, 1.5, 42.0];
let payload: ZBytes = z_serialize(&input);
let output: Vec<f32> = z_deserialize(&payload).unwrap();
```

`zenoh-ext` serialization doesn't pretend to cover all use cases, as it is just one available choice among other serialization formats like JSON, Protobuf, CBOR, etc. In the end, Zenoh will just send and receive payload raw bytes independently of the serialization used.  

NOTE: ⚠️ Serialization of `Vec<u8>` is not the same as creating a `ZBytes` from a `Vec<u8>`: the resulting `ZBytes` are different, and serialization doesn't take ownership of the bytes.

## Encoding

`Encoding` has been reworked. 
Zenoh does not impose any encoding requirement on the user, nor does it operate on it. 
It can be thought of as optional metadata, carried over by Zenoh in such a way that the end user’s application may perform different operations based on encoding.
We have expanded our list of pre-defined encoding types from Zenoh 0.11.0 for user convenience.
The module path and name of the encoding have also changed.

- Zenoh 0.11.x

```rust
use zenoh::prelude::KnownEncoding;

session
    .put(&key_expr, payload)
    .encoding(KnownEncoding::AppOctetStream)
    .res()
    .await
    .unwrap();
```

- Zenoh 1.0.0

```rust
use zenoh::encoding::Encoding;
session
    .put(&key_expr, payload)
    .encoding(Encoding::APPLICATION_OCTET_STREAM)
    .await
    .unwrap();
```

Users can also define their own encoding scheme that does not need to be based on the pre-defined variants.

```rust
let encoding = Encoding::from("pointcloud/LAS");
```

Because encoding is now optional for `put`, `Publisher` can be declared with a default encoding, which will be used in every `Publisher::put`.

```rust
let publisher = session.declare_publisher("my/keyepxr").encoding(Encoding::APPLICATION_JSON).await.unwrap();
// default encoding from publisher `application/json`
publisher.put(serde_json::to_vec(json!({"key", "value"})).unwrap()).await.unwrap();
```

## Attachment

In Zenoh 0.11.x, the `AttachmentBuilder` was required to create an attachment. 
In Zenoh 1.0.0, we have removed `AttachmentBuilder`, and an attachment can be created from anything that implements `Into<ZBytes>`

- Zenoh 0.11.x

```rust
let mut attachment = AttachmentBuilder::new();
attachment.insert("key1", "value1");
attachment.insert("key2", "value2");
publisher.put(payload)
		  	 .with_attachment(attachment.build())
		  	 .res()
		  	 .await
		  	 .unwrap();
```

- Zenoh 1.0.0

```rust
// Difference 1: No AttachmentBuilder anymore
//               Accept any type which can be transformed into ZBytes
let mut hashmap = HashMap::new();
hashmap.insert(String::from("key1"), String::from("value1"));
hashmap.insert(String::from("key2"), String::from("value2"));
let the_attachment = ZBytes::from(&hashmap);
// Difference 2: no with_attachment()
publisher
    .put(payload)
    .attachment(the_attachment)
    .await
    .unwrap();
```

##  API changes in Query & Queryable

Query and Queryable have been slightly reworked. 

For the API replying to a `Query` from a `Queryable` declared on a session: 
The reply function has been split into 3 separate functions variants depending on the type of reply the user wants to send.

- Zenoh 0.11.x

```rust
let reply_ok = Ok(Sample::new(key_expr.clone(), payload.clone())); // Success
query.reply(reply_ok).res().await.unwrap();
// or 
let reply_err = Err(Value::from(payload.clone()));                 // Failure
query.reply(reply_err).res().await.unwrap();
```

- Zenoh 1.0.0

```rust
// No need to send Result
// For sending Succesful Reply to Query
query.reply(key_expr.clone(), payload.clone()).await.unwrap();  // Success
// For sending Error Reply to Query
query.reply_err(payload.clone()).await.unwrap();                // Failure
// For sending Delete reply to Query (Sample Kind = Delete)
query.reply_del(payload.clone()).await.unwrap();                // Delete (Success)
```

For how a Get `Query` receives the reply: 
use `result()` on the `Reply` to get the `&Sample`, 
or `into_result` to take the ownership of the `Sample`.

 `Ok` variant replies, will return `Sample`. 
 `Err` variant replies, will return `ReplyError`

- Zenoh 0.11.x

```rust
while let Ok(reply) = replies.recv_async().await {
    match reply.sample {  // sample should be Result<Sample, Value>
        Ok(sample) => println!(
            ">> Received ('{}': '{}')",
            sample.key_expr.as_str(),
            sample.value,
        ),
        Err(value_err) => println!("{}", String::try_from(&value_err).unwrap()),
    }
}
```

- Zenoh 1.0.0

```rust
while let Ok(reply) = replies.recv_async().await {
    // Difference 1: using result() to get Result<&Sample, &ReplyError>
    match reply.result() {
        Ok(sample) => {
            println!(
                ">> Received ('{}': '{}')",
                sample.key_expr().as_str(),
                // Difference 2: payload() instead of value
                sample.payload().deserialize::<String>().unwrap()
            );
        }
        // Difference 3: ReplyError instead of Value
        Err(err) => {
            println!("{}", err.payload().deserialize::<String>().unwrap());
        }
    }
}
```

We have also added the ability to get underlying Handlers from `Queryables`, so that users have direct acces to the receiver of the data channel. 

```rust
let queryable = session
    .declare_queryable(&key_expr)
    .await
    .unwrap();

let handler: &Receiver<Query> = queryable.handler();
// or mutable handler
let mut_handler:&mut Receiver<Query> = queryable.handler_mut();
```

## Use accessors to get private members

We encapsulate members of structs, and they can’t be accessed directly now. 
The only way to access Struct values is to use the getter function associated with them. 
Let’s take the subscriber as an example here.

- Zenoh 0.11.x

```rust
while let Ok(sample) = subscriber.recv_async().await {
    println!(
        ">> [Subscriber] Received {} ('{}': '{}')",
        sample.kind,
        sample.key_expr.as_str(),
        sample.value
    );
}
```

- Zenoh 1.0.0

```rust
while let Ok(sample) = subscriber.recv_async().await {
    println!(
        ">> [Subscriber] Received {} ('{}': '{:?}')",
        sample.kind(),
        sample.key_expr().as_str(),
        sample.payload()  // Ignore the deserialization
    );
}
```

## Support RingChannel to receive data

Besides using a callback to receive data, we can also receive the data from a default FIFO channel. However, sometimes we only care about the latest data and want to discard the oldest data. 
We can use `RingChannel` to get this behaviour.
You can take a look at the complete code in `examples/examples/z_pull.rs`.

```rust
let subscriber = session
    .declare_subscriber(&key_expr)
    .with(RingChannel::new(size))
    .await
    .unwrap();
```

⚠️ Note:  We **no longer** support **Pull** mode in Zenoh

To get the same behavior of a Zenoh 0.11.0 `PullSubscriber`, please make use of a `RingChannel` an example of this is illustrated in `z_pull.rs`.

## Timestamps

We now tie generating a timestamp to a Zenoh session, with the timestamp inheriting the `ZenohID` of the session.

Note that a Zenoh session will only be able to generate a timestamp if the `timestamping` configuration option is enabled.

- Zenoh 0.11.x

```rust
let timestamp : Timestamp =  zenoh::time::new_reception_timestamp();
```

- Zenoh 1.0.0

```rust
let session: Session = zenoh::open();
// If the `timestamping` configuration is disabled, this call will return `None`.
let timestamp: Option<Timestamp> = session::new_timestamp();
```

This will affect user-created plugins and applications that need to generate timestamps.

## Feature Flags

Removed:

- `complete_n`: due to a Legacy code cleanup

## Storages

Zenoh 1.0.0 introduced the possibility for Zenoh nodes configured in a mode other than `router` to load plugins.

A, somehow, implicit assumption that dictated the behaviour of storages is that the Zenoh node loading them **has to add a timestamp to any received publication that did not have one**. This functionality is controlled by the `timestamping` configuration option.

Until Zenoh 1.0.0 this assumption held true as only a router could load storage and the default configuration for a router enables `timestamping`. However, in Zenoh 1.0.0 nodes configured in `client` & `peer` mode can load storage and *their default configuration disables `timestamping`*.

⚠️ The `storage-manager` will fail to launch if the `timestamping` configuration option is disabled.