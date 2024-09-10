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

This applies to all the builders that were in 0.11.0. Generally all the parameters on each builder now have their corresponding argument available in the functions.

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

Up until 0.11.0, it was up to the user to keep track of their variable declarations to keep them alive, because once the variable declarations were garbage collected, the declarations was closed. This was because each Kotlin variable declaration is associated with a native Rust instance, and in order to avoid leaking the memory of that Rust instance, it was necessary to free it upon dropping the declaration instance. However, this behavior could be counterintuitive, as users were expecting the declaration to keep running despite losing track of the reference to it.

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

The `Config` class has been modified

- `Config.default()`: returns the default config
- `Config.fromFile(file: File): Result<Config>`: allows to load a config file.
- `Config.fromPath(path: Path): Result<Config>`: allows to load a config file from a path.
- `Config.fromJson(json: String): Result<Config>`: loads the config from a string literal with json format
- `Config.fromJson5(json5: String): Result<Config>`: loads the config from a string literal with json5 format
- `Config.fromYaml(yaml: String): Result<Config>`: loads the config from a string literal with yaml format
- `Config.fromJsonElement(jsonElement: JsonElement): Result<Config>`: loads the config from a kotlin JsonElement.

In case of failure loading the config, a `Result.Error` is returned.

## Reliability

The `Reliability` config parameter used when declaring a `Subscriber` has been moved. It now must be specified when declaring a `Publisher` or when performing a `Put` or a `Delete` operaton.

## ZBytes serialization / deserialization & replacement of Value

We have created a new abstraction with the name of `ZBytes`. This class represents the bytes received through the Zenoh network. This new approach has the following implications:

- `Attachment` class is replaced by `ZBytes`.
- `Value` is replaced by the combination of `ZBytes` and `Encoding`.
- Replacing `ByteArray` to represent payloads

With `ZBytes` we have also introduced a Serialization and Deserialization for conveinent conversion between `ZBytes` and Kotlin types.

### Serialization

We can serialize primitive types into a `ZBytes` instance, that is, converting the data into bytes processed by the zenoh network:

#### Primitive types

The following types support serialization:

 * Numeric: `Byte`, `Short`, `Int`, `Long`, `Float` and `Double`.
 * `String`
 * `ByteArray`

 For the primitive types, there are three ways to serialize them into a `ZBytes`, for instance let's suppose
 we want to serialize an `Int`:

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
 This approach works as well for the other aforementioned types.

 Using `into()` or `from()` guarantees successful serialization for implemented types.  
 Using `serialize` requires a generic parameter, and returns a Result, i.e. it can fail based in the type passed in and contents of the input parameter.

 #### Lists

 Lists are supported, but they must be either:
 - List of `Number` : (`Byte`, `Short`, `Int`, `Long`, `Float`, `Double`)
 - List of `String`
 - List of `ByteArray`
 - List of `IntoZBytes`

 The serialize syntax must be used:
 ```kotlin
 val myList = listOf(1, 2, 5, 8, 13, 21)
 val zbytes = ZBytes.serialize<List<Int>>(myList).getOrThrow()
 ```

 #### Maps

 Maps are supported as well, with the restriction that their inner types must supported primitives:
 - `Number`
 - `String`
 - `ByteArray`
 - `IntoZBytes`

 ```kotlin
 val myMap: Map<String, Int> = mapOf("foo" to 1, "bar" to 2)
 val zbytes = ZBytes.serialize<Map<String, Int>>(myMap).getOrThrow()
 ```

 ### Deserialization

 #### Primitive types

 * Numeric: `Byte`, `Short`, `Int`, `Long`, `Float` and `Double`
 * `String`
 * `ByteArray`

 Example:

 For these primitive types, you can use the functions `to<Type>`, that is
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

 Lists are supported, but they must be deserialized into inner primitive types:
 - List of `Number` (`Byte`, `Short`, `Int`, `Long`, `Float` or `Double`)
 - List of `String`
 - List of `ByteArray`

 To deserialize into a list, use the deserialize syntax as follows:
 ```kotlin
 val inputList = listOf("sample1", "sample2", "sample3")
 payload = ZBytes.serialize(inputList).getOrThrow()
 val outputList = payload.deserialize<List<String>>().getOrThrow()
 ```

 #### Maps

 Maps are supported as well, with the restriction that their inner types must be one of the following:
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

 For a user defined class, the class must implement the `IntoZBytes` interface.
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

 Implementing the `IntoZBytes` interface on a class allows serializing lists and maps of that type, for instance:  
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
 of a `Map<KType, KFunction1<ZBytes, Any>>` map.  

 For instance, the previous implementation of our example Foo class.
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
