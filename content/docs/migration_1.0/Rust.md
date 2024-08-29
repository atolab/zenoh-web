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

## The changes to sync and async

In the previous version of Zenoh, we needed to use different module paths for the synchronous and asynchronous API.

Now both API are exposed behind `use zenoh::prelude::*`:

- Zenoh 0.11.x

```rust
// async
use zenoh::prelude::r#async::*;
// sync
use zenoh::prelude::sync::*;
```

- Zenoh 1.0.0

```rust
use zenoh::prelude::*;
```

Another big difference is that we have removed the need for `res()` in our asynchronous API.  
More inline with Rust naming, `wait()` is used for synchronous API, 
and `await` is used for the Asynchronous API. 
To make the migration easier, there is a deprecation prompt if you use the old API convention.

- Zenoh 0.11.x

```rust
// (deprecated) async
let session = zenoh::open(config).res().await.unwrap();
let publisher = session.declare_publisher(&key_expr).res().await.unwrap();
put.res().await.unwrap();
// (deprecated) sync
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
let session = zenoh::open(config).wait().unwrap();
let publisher = session.declare_publisher(&key_expr).wait().unwrap();
publisher.put(buf).wait().unwrap();
```

## Value is gone, long live ZBytes

We have replaced `Value` with `ZBytes` and `Encoding` , and added a number of conversion implementations such that user structs can be serialized into `ZBytes`, sent via Zenoh, and de-serialized from `ZBytes` with ease.

This is facilitated through the `ZSerde` struct, and implementations of the traits
`zenoh::bytes::Deserialize` and `zenoh::bytes::Serialize`.

We provide implementations of Zenoh’s aforementioned `Deserialize` and `Serialize` traits for primitive Rust types, Rust’s `Vec`, the `Value` type exposed by `Serde`'s various libraries as well as an example of `Protobuf` ’s `prost::Message` type. 

You can look at a full set of examples in `examples/examples/z_bytes.rs`.

NOTE: ⚠️ `ZSerde` is not the only serializer/deserializer users can make use of, nor a limitation to the types supported by Zenoh. Users are free to use whichever serializer/deserializer they wish!
- Zenoh 0.11.x

```rust
let sample = subscriber.recv_async().await.unwrap();
let value: Value = sample.value;
let the_string: String = value.try_into().unwrap();
```

- Zenoh 1.0.0

```rust
let sample = subscriber.recv_async().await.unwrap();
let zbytes: ZBytes = sample.payload();
let the_string: String = zbytes.deserialize::<String>().unwrap();
```

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