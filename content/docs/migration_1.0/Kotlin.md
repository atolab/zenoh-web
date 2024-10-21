---
title: "Kotlin"
weight : 6800
menu:
  docs:
    parent: migration_1.0
---

# Kotlin API migration guide for 1.0.0

The API has been extensively modified with the following goals in mind:
- Enhancing the API to be more idiomatic to Kotlin.
- Match the API rework done through the Rust Zenoh API.
- Abstracting users from the underlying native mechanisms

## Default arguments

Throughout the 0.11.0 API we were exposing builders, however we decided to replace all of them with the more Kotlin-idiomatic way of the default arguments.

For instance in 0.11.0:

```kotlin
session.put(keyExpr, value)
  .congestionControl(CongestionControl.BLOCK)
  .priority(Priority.REALTIME)
  .res()
```

Becomes the following in 1.0.0:
```kotlin
session.put(keyExpr, 
  value, 
  congestionControl = CongestionControl.BLOCK, 
  priority = Priority.REALTIME,
)
```

_Notice that the `.res()` function has been removed. Now functions exposed by the API execute imediately, without the need of `.res()`. i.e. When doing `session.put(...)` the put is run immediately._ 

This applies to all the builders being present on 0.11.0. Generally [^1] all the parameters present on each builder have now their corresponding argument available in the functions.

[^1]: Some parameters have been either moved (e.g. `Reliability`) or removed (e.g. `SampleKind`).


## Session opening

`Session.open(config: Config)` has now been replaced with `Zenoh.open(config: Config)`. 

## Encoding rework

The `Encoding` class has been modified. In 0.11.0. it had the signature
```kotlin
class Encoding(val knownEncoding: KnownEncoding, val suffix: String = "")
```
where `KnownEncoding` was an enum.

In 0.11.0. an encoding instance would be created as follows:
```kotlin
val encoding = Encoding(knownEncoding = KnownEncoding.TEXT_JSON)
```

In 1.0.0. we have implemented the following changes:
- `KnownEncoding` enum is removed, instead we provide static `Encoding` instances containing an ID and a description.
- Custom encodings can be created
- The list of pre-defined encodings has been expanded.

In 1.0.0. the previous example would instead now become:

```kotlin
val encoding = Encoding.TEXT_JSON
```

Custom encodings can be created by specifying an `id` and `suffix` string:

```kotlin
val encoding = Encoding(id = 123, suffix = "example suffix")
```

## Session-managed declarations

Up until 0.11.0, it was up to the user to keep track of their variable declarations to keep them alive, because once the variable declarations were garbage collected, the declarations were closed. This was because each Kotlin variable declaration is associated with a native Rust instance, and in order to avoid leaking the memory of that Rust instance, it was necessary to free it upon dropping the declaration instance. However, this behavior could be counterintuitive, as users were expecting the declaration to keep running despite losing track of the reference to it.

In this release we introduce a change in which any session declaration is internally associated to the session from which it was declared. Users may want to keep a reference to the declaration in case they want to undeclare it earlier before the session is closed, otherwise, the declaration is kept alive.

For instance:

```kotlin
val subscriber = session.declareSubscriber("A/B/C".intoKeyExpr(), 
  callback = { sample -> println("Receiving sample on 'A/B/C': ${sample.payload}") }).getOrThrow()

session.declareSubscriber("A/C/D".intoKeyExpr(), 
  callback = { sample -> println("Receiving sample on 'A/C/D': ${sample.payload}") }) // No variable is associated to the declared session, on 0.11.0 it would have been instantanely dropped
```

Therfore, when receiving a 'hello' message on `A/**` we would still see:

```
>> Receiving sample on 'A/B/C': hello
>> Receiving sample on 'A/C/D': hello
```
since both declarations are still alive.

Now the question is, for how long? What happens first, either when:
- you call `undeclare()` or `close()` to the declaration 
- the session is closed, then all the associated declarations are automatically undeclared.

## Key Expression rework

KeyExpr instances are not bound to a native key expression anymore, unless they are declared from a session. It is safe to drop the reference to the key expression instance, but the memory management associated to a key expression will differ:  
- If the KeyExpr was not declared from a session, then the garbage collector simply claims back the memory.   
- If it was declared from a session, then the session keeps track of it and frees the native memory upon closing the session.  

Declaring a KeyExpr on a session results in better performance, since the session is informed that we intend to use a key expression repeatedly.
We also associate a native key expression to a Kotlin key expression instance, avoiding copies.

## Config loading

When opening a session, it's now mandatory to provide a configuration to the session, even for a default config:
```kotlin
val config = Config.default()
Zenoh.open(config).onSuccess {
  // ...

}
```

The `Config` class has been modified

- `Config.default()`: returns the default config
- `Config.fromFile(file: File): Result<Config>`: allows to load a config file.
- `Config.fromPath(path: Path): Result<Config>`: allows to load a config file from a path.
- `Config.fromJson(json: String): Result<Config>`: loads the config from a string literal with json format
- `Config.fromJson5(json5: String): Result<Config>`: loads the config from a string literal with json5 format
- `Config.fromYaml(yaml: String): Result<Config>`: loads the config from a string literal with yaml format
- `Config.fromJsonElement(jsonElement: JsonElement): Result<Config>`: loads the config from a kotlin JsonElement.

In case of failure loading the config, a `Result.Error` is returned.

## Packages rework

The package structure of the API is now aligned with Zenoh Rust package structure.

Changes:

- Removing the "prelude" package
- QoS package now contains:
  - `CongestionCOntrol`
  - `Priority`
  - `Reliability`
  - `QoS`
- Bytes package is created with:
  - `ZBytes`, `IntoZBytes`, `Encoding`
- Config package:
  - `Config`, `ZenohId`
- Session package:
  - `SessionInfo`
- Query package:
  - contains `Query` and `Queryable`
  - removing queryable package

## Reliability

The `Reliability` config parameter used on when declaring a subscriber, has been moved. It now must be specified when declaring a `Publisher` or when performing a `Put` or a `Delete` operaton.


## Logging

There are two main changes regarding logging, the interface and the mechanism.

Lets look at the following example, where we want to run the ZPub example with debug logging. On 1.0.0 we'll do:

```
RUST_LOG=debug gradle ZPub
```

If we wanted to enable debug logging and tracing for some specific package, for instance `zenoh::net::routing`, we'd do:

```
RUST_LOG=debug,zenoh::net::routing=trace gradle ZPub
```

However, this is not enabled by default.
In order to enable logging, one of these newly provided functions must be called:

```kotlin
Zenoh.tryInitLogFromEnv()
```

and 

```kotlin
Zenoh.initLogFromEnvOr(fallbackFilter: String): Result<Unit>
```

This last function allows to programmatically specify the logs configuration if not provided as an environment variable.

## ZBytes (de)serialization & replacement of Value

We have created a new abstraction with the name of `ZBytes`. This class represents the bytes received through the Zenoh network. This new approach has the following implications:

- `Attachment` class is replaced by `ZBytes`.
- `Value` is replaced by the combination of `ZBytes` and `Encoding`.
- Replacing `ByteArray` to represent payloads

With `ZBytes` we have also introduced a Serialization and Deserialization for conveinent conversion between `ZBytes` and Kotlin types.

### (DE)Serialization

ZBytes can be serialized and deserialized into/from a combination of the following types:
- Boolean
- Byte
- Short
- Int
- Long
- Float
- Double
- UByte
- UShort
- UInt
- ULong
- List
- String
- ByteArray
- Map
- Pair
- Triple

For `List`, `Pair` and `Triple`, the inner types can be a combination of the above types, including themselves.
These serialization and deserialization utilities can be used across the Zenoh ecosystem with Zenoh
versions based on other supported languages such as Rust, Python, C and C++.
This works when the types are equivalent (a `Byte` corresponds to an `i8` in Rust, a `Short` to an `i16`, etc).

#### Examples

```kotlin
/** String serialization example */
val originalString: String = "Hello, Kotlin!"
val stringZBytes = zSerialize(originalString).getOrThrow()
val deserializedString = zDeserialize<String>(stringZBytes).getOrThrow()
check(originalString == deserializedString)

/** Map serialization example */
val originalMap: Map<String, String> = mapOf("key1" to "value1", "key2" to "value2")
val mapZBytes = zSerialize(originalMap).getOrThrow()
val deserializedMap = zDeserialize<Map<String, String>>(mapZBytes).getOrThrow()
check(originalMap == deserializedMap)

/** List serialization example */
val originalList: List<String> = listOf("apple", "banana", "cherry")
val listZBytes = zSerialize(originalList).getOrThrow()
val deserializedList = zDeserialize<List<String>>(listZBytes).getOrThrow()
check(originalList == deserializedList)
```

Checkout the [ZBytes examples](https://github.com/eclipse-zenoh/zenoh-kotlin/blob/main/examples/src/main/kotlin/io.zenoh/ZBytes.kt) for more examples.

## Reply handling

Previously on 0.11.0 we were exposing the following classes:

- `Reply.Success`
- `Reply.Error`

In 0.11.0 executing a `get` on a session was done in the following way. 

```kotlin
session.get(selector).res()
  .onSuccess { receiver ->
      runBlocking {
          for (reply in receiver!!) {
              when (reply) {
                  is Reply.Success -> {println("Received ('${reply.sample.keyExpr}': '${reply.sample.value}')")}
                  is Reply.Error -> println("Received (ERROR: '${reply.error}')")
              }
          }
      }
  }
```

Now, in 1.0.0, the `Reply` instances contain a `result` attribute, which is of type `Result<Sample>`. 

```kotlin
session.get(selector, channel = Channel())
  .onSuccess { channelReceiver ->
      runBlocking {
          for (reply in channelReceiver) {
              reply.result.onSuccess { sample ->
                  when (sample.kind) {
                      SampleKind.PUT -> println("Received ('${sample.keyExpr}': '${sample.payload}')")
                      SampleKind.DELETE -> println("Received (DELETE '${sample.keyExpr}')")
                  }
              }.onFailure { error ->
                  println("Received (ERROR: '${error.message}')")
              }
          }
      }
  }
```
