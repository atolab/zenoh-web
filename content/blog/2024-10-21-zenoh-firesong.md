---
date: "2024-10-21"
title: 'Ladies and gentlemen: habemus Zenoh 1.0.0 "Firesong"!'
description: "21st October 2024"
menu: "blog"
weight: 20241021
---

As a result of an incredible effort from the whole Zenoh team and Zenoh community, we can finally announce that Zenoh 1.0.0 *Firesong* is out! 

This release comes with a lot of features and improvements:
- API stabilization. Great attention has been given to the API, its revision and rework to provide the necessary level of stability and future extensibility.
- The very first alpha version of the new TypeScript API.
- A full rework of the Shared Memory subsystem in Zenoh, with a new API and more supported topologies.
- Improved batching and jitter performance for high frequency publications.
- Improved protocol for write-side filtering.

Let us take a closer look at what Zenoh 1.0.0 brings to the table. 

---

# API improvement

Zenoh's API has been improved in terms of ergonomics, clarity, and composability for future extensibility! 
The following sections highlight the main changes of the API in the various language bindings. 
The full migration guide of each language is available [here](https://zenoh.io/docs/migration_1.0/concepts/).

## Accessors

To better separate the public API from the internal implementation, we have introduced the accessor pattern in the Zenoh API across all language bindings. 
See an example in Rust below. 
Please note that the same approach would apply to all Zenoh APIs.

Zenoh 0.11.0:
```rust
while let Ok(sample) = subscriber.recv_async().await {
    println!(
        ">> [Subscriber] Received {} ('{}': '{}')",
        sample.kind,
        sample.key_expr.as_str(),
    );
}
```

Zenoh 1.0.0:
```rust
while let Ok(sample) = subscriber.recv_async().await {
    println!(
        ">> [Subscriber] Received {} ('{}': '{}')",
        sample.kind(),
        sample.key_expr().as_str(),
    );
}
```

## Value is gone, long live ZBytes

`Value` was a type that contained a payload (`ZBuf`) and some metadata about the `Encoding`. It was generally accepted in functions like `put()` and `reply()`. 
Zenoh 1.0.0 deprecates `Value` in favour of explicitly separating `ZBytes` and `Encoding` in the API. This has the benefit of improving API and network overhead. 

`ZBytes` is the core type for raw data representation in Zenoh. 
All API has been reworked to accept `ZBytes` or a type that can be transparantely converted into a `ZBytes`, such as a string.
Sample’s payloads are now `ZBytes`. `Publisher`, `Queryable` and `Subscriber` now expect `ZBytes` for all their interfaces.
The Attachment API also now accepts `ZBytes`.

Zenoh 0.11.0:
```rust
// Publisher
// “My Value” is converted in bytes and the “string” encoding was automatically set
session.put(“test/foo”, “My Value”).res().await.unwrap(); 

// Subscriber
// Data can be accessed directly and encoding metadata is not required to be checked
// Some bandwidth was wasted for nothing
let string = String::from_utf8_lossy(sample.value.contiguous());
```

Zenoh 1.0.0:
```rust
// Publisher
// “My Value” is a string that can be diretly converted in bytes, no encoding is automatically set
session.put(“test/foo”, “My Value”).attachment(“My attachment”).await.unwrap(); 

// Subscriber
// Convert data type using
let value = sample.payload().try_to_string().unwrap();
let attach = sample.attachment().try_to_string().unwrap();
```

It is worth highlighting that Zenoh semantics and protocol take care of sending and receiving bytes without restricting the actual data types.
Nonetheless, in the spirit of always improving user's life with a simple out-of-the-box experience, we have added a number of optional serializer/deserializer to deal with language primitive types (e.g., integeres, floats, tuples, etc.).
This serializer/deserializer is NOT by any means the only serializer/deserializer users can use nor a limitation to the types supported by Zenoh. 
Users are free and encouraged to use any serializer/deserializer of their choice like JSON, protobuf, bincode, flatbuffers, etc. 

```rust
use zenoh_ext::{z_deserialize, z_serialize};

// Numeric types: u8, u16, u32, u128, i8, i16, i32, i128, f32, f64
let input = 1234_u32;
let payload = z_serialize(&input);
let output: u32 = z_deserialize(&payload).unwrap();
assert_eq!(input, output);

// Vec
let input = vec![0.0f32, 1.5, 42.0];
let payload = z_serialize(&input);
let output: Vec<f32> = z_deserialize(&payload).unwrap();
assert_eq!(input, output);

// HashMap
let mut input: HashMap<u32, String> = HashMap::new();
input.insert(0, String::from("abc"));
input.insert(1, String::from("def"));
let payload = z_serialize(&input);
let output: HashMap<u32, String> = z_deserialize(&payload).unwrap();
assert_eq!(input, output);

// Tuple
let input = (0.42f64, "string".to_string());
let payload = z_serialize(&input);
let output: (f64, String) = z_deserialize(&payload).unwrap();
assert_eq!(input, output);

// Array (handled as variable-length sequence, not as tuple)
let input = [0.0f32, 1.5, 42.0];
let payload = z_serialize(&input);
let output: [f32; 3] = z_deserialize(&payload).unwrap();
assert_eq!(input, output);
// can also be deserialized as a vec
let output: Vec<f32> = z_deserialize(&payload).unwrap();
assert_eq!(input.as_slice(), output);
```

You can have a look at the [z_bytes example](https://github.com/eclipse-zenoh/zenoh/blob/main/examples/examples/z_bytes.rs) for additional examples.

## Encoding

`Encoding` has been reworked, moving away from enumerables to now accepting strings.
While Zenoh does not impose any `Encoding` requirement on the user, providing an `Encoding` can offer automatic wire-level optimization for known types.  
For the user defined `Encoding`, it can be thought of as optional metadata, carried over by Zenoh in such a way that the end user’s application may perform different operations based on `Encoding`.  
We have expanded our list of predefined encoding types from Zenoh 0.11.0 to include variants for numerous IANA standard encodings, including but not limited to  `video/x` , `application/x`, `text/x`, `image/x` and `audio/x` encoding families, as well as an encoding family specific to Zenoh defined by the prefix `zenoh/x` .   
Users can also define their own encoding scheme that does not need to be based on the predefined IANA variants. 

```rust
// Publisher
// Encoding::TEXT_PLAIN is a convenience constant equivalent to “text/plain”
session.put(“test/foo”, “My Value”).encoding(Encoding::TEXT_PLAIN).await.unwrap();

// Subscriber
if sample.encoding() == Encoding::TEXT_PLAIN {
    let s = sample.payload().try_to_string().unwrap()
}
```

Example of using custom encoding:

```rust
// Publisher
// Encoding::TEXT_PLAIN is a convenience constant equivalent to “text/plain”
session.put(“test/foo”, vec![0u8; 64]).encoding(“my_encoding”).await.unwrap();

// Subscriber
if &sample.encoding().to_string() == “my_encoding” {
    let reader = sample.payload().reader();
    // Deserialize the type according to the encoding
}
```

## Query & Queryable
The `reply` method of a `Queryable` has gained two variants: 
- `reply_del` to indicate that a deletion should be performed.
- `reply_err` to indicate that an error occurred.   

Additionally, these variants behave similarly to `put` and `del`, providing improved ergonomics.

We have added the ability to get the underlying `Handler` of a Queryable as well.

```rust
// Queryable
while let Ok(query) = queryable.recv_async().await {
    query
        .reply(key_expr.clone(), payload.clone())
         // Or reply with a delete
         // .reply_del(key_expr.clone())
         // Or reply with an error
         // .reply_err(“My error”)
        .await
        .unwrap();
}

// Querier
let replies = session.get(“test/foo”).await.unwrap();

while let Ok(reply) = replies.recv_async().await {
    match reply.result() {
        Ok(sample) => match sample.kind() {
            SampleKind::Put => { /* Handle .reply() */ },
            SampleKind::Del => { /* Handle .reply_del() */ },
        },
        Err(err) => { /* Handle .reply_err() and any infrastructure error */  }
    }
}
```

## Timestamps

Zenoh timestamps are based on an [Unique Hybrid Logical Clock](https://github.com/atolab/uhlc-rs), which is largely used in the Zenoh Storage Alignment protocol. One of the requirements for the alignment protocol is to be able to uniquely identify who published the data and consequently who generated the timestamp. Zenoh 0.11.0 exposed a function to generate timestamps outside of a session, which could be used in a Zenoh system. Due to our efforts to improve the storage replication logic, now timestamps are generated from a session, with the timestamp inheriting the `ZenohID` of the session. Users are not expected to generate timestamps by themselves but rather to rely on the [timestamping functionality](https://github.com/eclipse-zenoh/zenoh/blob/6b7ec55911ce85a243c6eae857cbddd7ab1d0021/DEFAULT_CONFIG.json5#L133) of Zenoh.

## Pull Subscriber has been removed, ring channel has been added

A pull subscriber was mainly used to pull the latest data available for a given subscriber. 
In Zenoh 1.0.0, we have removed the pull subscriber as part of some rework to streamline the API and the protocol. 
This leads to the fact that only one type of subscriber exists. 
However, to keep the functionality of being able to pull the last data, we have added a `RingChannel` for the subscriber (as well as for the queryable and get) that can be used to get a similar behaviour. 
Once full, the `RingChannel` will replace older data with most recent ones. This contrasts with the `FIFOChannel`, the default channel type used by Subscribers, which blocks once its buffer is full. 

Publisher example:
```rust
#[tokio::main]
async fn main() {
    let session = zenoh::open(Config::default()).await.unwrap();
    let publisher = session.declare_publisher(“test/foo”).await.unwrap();
    // Publish a message every second
    for idx in 0..10 {
        tokio::time::sleep(Duration::from_secs(1)).await;
        let buf = format!("[{idx:4}] Pub from Rust!");
        println!("{}", buf);
        publisher
            .put(buf)
            .await
            .unwrap();
    }
}
```

Subscriber example with FifoChannel (default):
```rust
#[tokio::main]
async fn main() {
    let session = zenoh::open(Config::default()).await.unwrap();
    let subscriber = session.declare_subscriber(“test/foo”).await.unwrap();
    while let Ok(sample) = subscriber.recv_async().await {
        let payload = sample
            .payload()
            .try_to_string()
            .unwrap_or_else(|e| format!("{}", e));
        println!(“{}”, payload); // No long computation
    }
}
```

Subscriber example with RingChannel:
```rust
#[tokio::main]
async fn main() {
    let session = zenoh::open(Config::default()).await.unwrap();
    let subscriber = session
             .declare_subscriber(“test/foo”)
             .with(RingChannel::new(5))
             .await
             .unwrap();
    while let Ok(sample) = subscriber.recv_async().await {
        let payload = sample
            .payload()
            .try_to_string()
            .unwrap_or_else(|e| format!("{}", e));
        println!(“{} Sleeping for 5s.”, payload); 
        tokio::time::sleep(Duration::from_secs(5)).await; // Long computation
    }
}
```

You can take a look at examples of usage in any language’s *examples/z_pull.x*.
The output of the publisher will look like this with a publication made every second. 
```sh
$ ./z_pub
[   0] Pub from Rust!
[   1] Pub from Rust!
[   2] Pub from Rust!
[   3] Pub from Rust!
[   4] Pub from Rust!
[   5] Pub from Rust!
[   6] Pub from Rust!
[   7] Pub from Rust!
[   8] Pub from Rust!
[   9] Pub from Rust!
[  10] Pub from Rust!
```

Since *z_sub* uses a `FifoChannel` then it will receive all samples. Please note that in the case of having a slow subscriber, back pressure could reach back the publisher. 
```sh
$ ./z_sub
[   0] Pub from Rust!
[   1] Pub from Rust!
[   2] Pub from Rust!
[   3] Pub from Rust!
[   4] Pub from Rust!
[   5] Pub from Rust!
[   6] Pub from Rust!
[   7] Pub from Rust!
[   8] Pub from Rust!
[   9] Pub from Rust!
[  10] Pub from Rust!
```

If you want the subscriber to receive and process the most recent samples, use the `RingChannel`. In this case, the subscriber will take 5 seconds to process a sample, not being able to keep up with the publication rate. Using the `RingChannel` will then allow you to drop old samples and get a more recent one. 
E.g. if you want to always process the most recent sample, you can set the size of the `RingChannel` to 1.
```sh
$ ./z_pull
[   0] Pub from Rust! Sleeping for 5s.
[   2] Pub from Rust! Sleeping for 5s. # Missed sample 1
[   7] Pub from Rust! Sleeping for 5s. # Missed sample 3, 4, 5, 6
[  12] Pub from Rust! Sleeping for 5s. # Missed sample 8, 9, 10, 11
```

# Bugfixes

- ...

And more. Checkout the [release changelog](... TO BE DONE ...) to check all the new features and bug fixes with their associated PR’s!

---

# What’s next?

Happy Hacking,

– The Zenoh Team

P.S. You can reach us out on Zenoh’s [Discord server](https://discord.com/invite/vSDSpqnbkm)!

![Guitar](../../img/20231003-blog-zenoh-dragonite/zenoh-on-fire.gif)
