---
title: "Kotlin"
weight : 6800
menu:
  docs:
    parent: migration_1.0
---

# Kotlin API migration guide for 1.0.0

The api has been extensively modified with the following goals in mind:
- Enhancing the API to be more idiomatic to Kotlin.
- Match the API rework done through the Rust Zenoh API.
- Abstracting users from the underlying native mechanisms

## Default arguments

Throughout the 0.11.0 API we were exposing builders, however we decided to replace all of them with the more Kotlin-idiomatic way of the default arguments.

For instance:

```kotlin
session.put(keyExpr, value)
  .congestionControl(CongestionControl.BLOCK)
  .priority(Priority.REALTIME)
  .res()
```

would now turn into:
```kotlin
session.put(keyExpr, 
  value, 
  congestionControl = CongestionControl.BLOCK, 
  priority = Priority.REALTIME,
)
```

_Notice that the `.res()` function is gone, now the functions behavior is more imperative, that is, when doing `session.put(...)` the put is launched immediately._ 

This applies to all the builders being present on 0.11.0. Generally [^1] all the parameters present on each builder have now their corresponding argument available in the functions.

[^1]: Some parameters have been either moved (e.g. `Reliability`) or removed (e.g. `SampleKind`).

## Encoding rework

The `Encoding` class has been modified. It previously had the signature
```kotlin
class Encoding(val knownEncoding: KnownEncoding, val suffix: String = "")
```

where `KnownEncoding` was an enum.

When creating an instance, we'd have done:
```kotlin
val encoding = Encoding(knownEncoding = KnownEncoding.TEXT_JSON)
```

Instead, on 1.0.0, we have provided the following changes:
- `KnownEncoding` enum is removed, instead we provide static `Encoding` instances containing an ID and a description.
- Custom encodings can be created
- Also the list of provided encodings has been expanded.

Therefore, the previous example would instead be now:

```kotlin
val encoding = Encoding.TEXT_JSON
```

Custom encodings can be created with

```kotlin
val encoding = Encoding(id = 123, suffix = "example suffix")
```
## Session-managed declarations

Up until 0.11.0, it was up to the user to keep track of their declarations to keep them alive, since when they were garbage collected, the declaration was closed. This had a reason to be, since each declaration is associated to a native instance, in order to avoid a native memory leak, it was necessary to free it upon dropping of the declaration instance. However, this behavior could be counterintuitive, sometimes, users were expecting the declaration to keep running despite loosing track of the reference to it.

In this release we introduce a change in which any session declaration is internally associated to the session from which it was declared. Users may want to keep a reference to the declaration in case they want to undeclare it earlier before the session is closed, otherwise, the declaration is kept alive.

For instance:

```kotlin
val subscriber = session.declareSubscriber("A/B/C".intoKeyExpr(), 
  callback = { sample -> println("Receiving sample on 'A/B/C': ${sample.payload}") }).getOrThrow()

session.declareSubscriber("A/C/D".intoKeyExpr(), 
  callback = { sample -> println("Receiving sample on 'A/C/D': ${sample.payload}") }) // No variable is associated to the declared session, on 0.11.0 it would have been instantanely dropped
```

Therfore, when doing a receiving a 'hello' message on `A/**` we'd still see:

```
>> Receiving sample on 'A/B/C': hello
>> Receiving sample on 'A/C/D': hello
```

since both declarations are still alive.

Now the question is, for how long? What happens first, either when:
- you call `undeclare()` or `close()` to the declaration 
- the session is closed, then all the associated declarations are automatically undeclared.

## KeyExpr rework

KeyExpr instances are not bound to a native key expression anymore, unless they are declared from a session. In any case, it's safe to drop the reference to the key expression instance, but the memory management associated to a key expression will differ: if it's not been declared from a session, then the garbage collector simply claims back the memory. If it was declared, then the session keeps track of it and frees the native memory upon closing the session.

What's the difference between declaring or not a key expression?
Declaring a key expression allows better performance, since the session is informed we intend to use a key expression repeatedly. We also associate a native key expression to a Kotlin key expression instance, avoiding copies.


## Config loading

When opening a session, it's now mandatory to provide a configuration to the session, even for a default config:
```kotlin
val config = Config.default()
Session.open(config).onSuccess {
  // ...

}
```

For this, the `Config` class has been modified

- `Config.default()`: returns the default config
- `Config.fromFile(file: File): Result<Config>`: allows to load a config file.
- `Config.fromPath(path: Path): Result<Config>`: allows to load a config file from a path.
- `Config.fromJson(json: String): Result<Config>`: loads the config from a string literal with json format
- `Config.fromJson5(json5: String): Result<Config>`: loads the config from a string literal with json5 format
- `Config.fromYaml(yaml: String): Result<Config>`: loads the config from a string literal with yaml format
- `Config.fromJsonElement(jsonElement: JsonElement): Result<Config>`: loads the config from a kotlin JsonElement.

In case of failure loading the config, a `Result.Error` is returned.

## Reliability

The `Reliability` config parameter used on when declaring a subscriber, has been moved. It now must be specified when declaring a `Publisher` or when performing a `Put` or a `Delete` operaton.

## ZBytes serialization / deserialization & replacement of Value

We have created a new abstraction with the name of `ZBytes`. This class represents the bytes received through the Zenoh network. This new approach has the following implications:

- `Attachment` class replacement.
- `Value` removal (instead use `ZBytes` and `Encoding`).
- Replacing `ByteArray` to represent payloads

Also, along with the `ZBytes` class, commes along the concepts of Serialization and Deserialization.

### Serialization

We can serialize certain types into a ZBytes instance, that is, converting the data into bytes processed by the zenoh network:

#### Raw types

The following raw types support serialization:

 * Numeric: Byte, Short, Int, Long, Float and Double.
 * String
 * ByteArray

 For the raw types, there are basically three ways to serialize them into a ZBytes, for instance let's suppose
 we want to serialize an `Int`, we could achieve it by:

 * using the `into()` syntax:
  ```kotlin
  val exampleInt: Int = 256
  val zbytes: ZBytes = exampleInt.into()
  ```

 * using the `from()` syntax:
  ```kotlin
  val exampleInt: Int = 256
  val zbytes: ZBytes = ZBytes.from(exampleInt)
  ```

 * using the serialize syntax:
 ```kotlin
 val exampleInt: Int = 256
 val zbytes: ZBytes = ZBytes.serialize<Int>(exampleInt).getOrThrow()
 ```
 This approach works as well for the other mentioned types.

 The difference using `into()` or `from()`, from using `serialize`, is that `serialize` returns a Result, meaning `into()` and `from()` guarantee the successful result.

 #### Lists

 Lists are supported, but they must be either:
 - List of `Number` (Byte, Short, Int, Long, Float or Double)
 - List of `String`
 - List of `ByteArray`
 - List of `IntoZBytes`

 The serialize syntax must be used:
 ```kotlin
 val myList = listOf(1, 2, 5, 8, 13, 21)
 val zbytes = ZBytes.serialize<List<Int>>(myList).getOrThrow()
 ```

 #### Maps

 Maps are supported as well, with the restriction that their inner types must be either:
 - `Number`
 - `String`
 - `ByteArray`
 - `IntoZBytes`

 ```kotlin
 val myMap: Map<String, Int> = mapOf("foo" to 1, "bar" to 2)
 val zbytes = ZBytes.serialize<Map<String, Int>>(myMap).getOrThrow()
 ```

 ### Deserialization

 #### Raw types

 * Numeric: `Byte`, `Short`, `Int`, `Long`, `Float` and `Double`
 * `String`
 * `ByteArray`

 Example:

 For these raw types, you can use the functions `to<Type>`, that is
 - `toByte`
 - `toShort`
 - `toInt`
 - `toLong`
 - `toDouble`
 - `toString`
 - `toByteArray`

 For instance, for an Int:
 ```kotlin
 val example: Int = 256
 val zbytes: ZBytes = exampleInt.into()
 val deserializedInt = zbytes.toInt()
 ```

 Alternatively, the deserialize syntax can be used as well:
 ```kotlin
 val exampleInt: Int = 256
 val zbytes: ZBytes = exampleInt.into()
 val deserializedInt = zbytes.deserialize<Int>().getOrThrow()
 ```

 #### Lists

 Lists are supported, but they must be deserialized either into a:
 - List of `Number` (Byte, Short, Int, Long, Float or Double)
 - List of `String`
 - List of `ByteArray`

 To deserialize into a list, we need to use the deserialize syntax as follows:
 ```kotlin
 val inputList = listOf("sample1", "sample2", "sample3")
 payload = ZBytes.serialize(inputList).getOrThrow()
 val outputList = payload.deserialize<List<String>>().getOrThrow()
 ```

 #### Maps

 Maps are supported as well, with the restriction that their inner types must be either:
 - `Number`
 - `String`
 - `ByteArray`

 ```kotlin
 val inputMap = mapOf("key1" to "value1", "key2" to "value2", "key3" to "value3")
 payload = ZBytes.serialize(inputMap).getOrThrow()
 val outputMap = payload.deserialize<Map<String, String>>().getOrThrow()
 check(inputMap == outputMap)
 ```

 ### Custom serialization and deserialization

 #### Serialization

 For custom serialization, classes to be serialized need to implement the `IntoZBytes` interface.
 For instance:

 ```kotlin
 class Foo(val content: String) : IntoZBytes {

   /*Inherits: IntoZBytes*/
   override fun into(): ZBytes = content.into()
 }
 ```

 This way, we can do:
 ```kotlin
 val foo = Foo("bar")
 val serialization = ZBytes.serialize<Foo>(foo).getOrThrow()
 ```

 Implementing the `IntoZBytes` interface on a class enables the possibility of serializing lists and maps
 of that type, for instance:
 ```kotlin
 val list = listOf(Foo("bar"), Foo("buz"), Foo("fizz"))
 val zbytes = ZBytes.serialize<List<Foo>>(list)
 ```

 #### Deserialization

 Regarding deserialization for custom objects, for the time being (this API will be expanded to
 provide further utilities) you need to manually convert the ZBytes into the type you want.

 ```kotlin
 val inputFoo = Foo("example")
 payload = ZBytes.serialize(inputFoo).getOrThrow()
 val outputFoo = Foo.from(payload)
 check(inputFoo == outputFoo)

 // List of Foo.
 val inputListFoo = inputList.map { value -> Foo(value) }
 payload = ZBytes.serialize<List<Foo>>(inputListFoo).getOrThrow()
 val outputListFoo = payload.deserialize<List<ZBytes>>().getOrThrow().map { zbytes -> Foo.from(zbytes) }
 check(inputListFoo == outputListFoo)

 // Map of Foo.
 val inputMapFoo = inputMap.map { (k, v) -> Foo(k) to Foo(v) }.toMap()
 payload = ZBytes.serialize<Map<Foo, Foo>>(inputMapFoo).getOrThrow()
 val outputMapFoo = payload.deserialize<Map<ZBytes, ZBytes>>().getOrThrow()
     .map { (key, value) -> Foo.from(key) to Foo.from(value) }.toMap()
 check(inputMapFoo == outputMapFoo)
 ```

 ##### Deserialization functions:

 As an alternative, the `deserialize` function admits an argument which by default is an emptyMap, consisting
 of a `Map<KType, KFunction1<ZBytes, Any>>` map. This allows to specify types in a map, associating
 functions for deserialization for each of the types in the map.

 For instance, let's stick to the previous implementation of our example Foo class.
 We could provide directly the deserialization function as follows:

 ```kotlin
 fun deserializeFoo(zbytes: ZBytes): Foo {
   return Foo(zbytes.toString())
 }

 val foo = Foo("bar")
 val zbytes = ZBytes.serialize<Foo>(foo)
 val deserialization = zbytes.deserialize<Foo>(mapOf(typeOf<Foo>() to ::deserializeFoo)).getOrThrow()
 ```

## Reply handling

Previously on 0.11.0 we were exposing the following classes:

- `Reply.Success`
- `Reply.Error`

We were having this interface (using the default channel receiver):

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

Now, the `Reply` instances contain a `result` attribute, which is of type `Result<Sample>`. Therefore, the updated code would be:

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
