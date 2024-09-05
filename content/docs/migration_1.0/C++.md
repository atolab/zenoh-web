---
title: "C++"
weight : 6300
menu:
  docs:
    parent: migration_1.0
---


Zenoh 1.0.0 brings a number of changes to the API, with a concentrated effort to bring the C++ API to more closely resemble the Rust API in usage.

The improvements we bring in this update include:

- A simpler organization of the Zenoh classes, abstracting away the notion of View and Closure.
- Improved and more flexible Error Handling through error codes and exceptions.
- Support for serialization of common types like strings, numbers, vectors through Codecs.
- Ability for users to define their own Codecs (for their own types or to overwrite the default one)!
- Improved stream handlers and callback support.
- Simpler attachment API.

Now that the *amuse bouche* is served,  let's get into the main course!

## Error Handling

In version 0.11.0 all Zenoh call failures were handled by either returning a `bool` value indicating success or failure (and probably returning an error code) or returning an `std::variant<ReturnType, ErrorMessage>`. For instance:

```cpp
std::variant<z::Config, ErrorMessage> config_client(const z::StrArrayView& peers);

bool put(const z::BytesView& payload, const z::PublisherPutOptions& options, ErrNo& error);

```

In 1.0.0, all functions that can fail on the Zenoh side now offer 2 options for error handling:

**A**. Exceptions

**B**. Error Codes 

Any function that can fail now accepts an optional parameter `ZError* err` pointer to the error code. If it is not provided (or set to `nullptr`), an instance of `ZException` will be thrown, otherwise the error code will be written into the `err` pointer. 

```cpp
static Config client(const std::vector<std::string>& peers, ZError* err = nullptr);
```

This also applies to constructors: if a failure occurs, either an exception is thrown or the error code is written to the provided pointer. In the latter case, the returned object will be in an "empty" state (i.e. converting it to a boolean returns `false`).

```cpp
Config config = Config::create_default();
// Receiving an error code
Zerror err = Z_OK;
auto session = Session::open(std::move(config), &err);
if (err != Z_OK) { // or alternatively if (!session)
  // handle failure
}

// Catching exception
Zenoh::session s(nullptr); // Empty session object
try {
  s = Session::open(std::move(config), &err);
} catch (const ZException& e) {
  // handle failure
}
```

All returned and `std::move`'d-in objects are guaranteed to be left in an “empty” state in case of function call failure.

## Serialization  and Deserialization

In version 0.11.0 it was only possible to send `std::string`/ `const char*` or `std::vector<uint8_t>` / `uint8_t*` using the `BytesView` class:

```cpp
publisher.put("my_payload");
```

In 1.0.0, the `BytesView` class is gone and we introduced the `Bytes` object which represents a serialized payload.

 

```cpp
void publish_data(const Publisher& publisher, const MyData& data) {
	publisher.put(Bytes<MyCodec>::serialize(data));
}

void receive_data(const Sample &sample) {
  std::cout <<"Received: " 
					  << sample.get_payload().deserialize<MyData, MyCodec>() 
            << "\n";
};
```

We added a default `ZenohCodec`, which provides default serialization / deserialization for common numerical types, strings, and containers:

```cpp
  // stream of bytes serialization
  std::vector<uint8_t> data = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  Bytes b = Bytes::serialize(data);
  assert(b.deserialize<std::vector<uint8_t>>() == data);
  
  // arithmetic type serialization
  double pi = 3.1415926;
  Bytes b = Bytes::serialize(pi);
  assert(b.deserialize<TYPE>() == pi);
  
  // Composite types serialization
  std::vector<float> v = {0.1f, .2f, 0.3f};
  auto b = Bytes::serialize(v);
  assert(b.deserialize<decltype(v)>() == v);
  
  std::map<std::string, std::deque<double>> m = {
    {"a", {0.5, 0.2}},
    {"b", {-123.45, 0.4}},
    {"abc", {3.1415926, -1.0} }
  };

  b = Bytes::serialize(m);
  assert(b.deserialize<decltype(m2)>() == m); 

  // alternatively serialize via move 
  // the string keys will not be copied in this case, but rather move into Bytes
  b_move = Bytes::serialize(std::move(m));
```

Please note that this serialization functionality is only provided for prototyping and demonstration purposes and, as such, might be less efficient than custom-written serialization methods.

Users can easily define their own serialization/deserialization functions by providing a custom codec:

```cpp
MyType my_data(...);
MyCodec my_codec(...);
Bytes b = Bytes::serialize(my_data, my_codec);
// or Bytes::serialize<MyCodec>(my_data) if MyCodec is a stateless codec with an empty constructor 
assert(b.deserialize<std::vector<MyType>>(my_codec) == my_data);
// or assert(b.deserialize<std::vector<MyType, MyCodec>>() == my_data); if MyCodec is a stateless with an empty constructor
```

For finer control on serialization / deserialization implementation `Bytes::Iterator`, `Bytes::Writer` and `Bytes::Reader` classes are also introduced.

A working example with custom-defined serialization / deserialization can be found here:

https://github.com/eclipse-zenoh/zenoh-cpp/blob/dev/1.0.0/examples/simple/universal/z_simple.cxx

## Stream Handlers and Callbacks

In version 0.11.0 stream handlers were only supported for `get` :

```cpp
// callback
session.get(keyexpr, "", {on_reply, on_done}, opts);

// stream handlers interface
auto [send, recv] = reply_fifo_new(16);
session.get(keyexpr, "", std::move(send), opts);

Reply reply(nullptr);
// blocking
for (recv(reply); reply.check(); recv(reply)) {
    auto sample = expect<Sample>(reply.get());
    std::cout << "Received ('" << sample.get_keyexpr().as_string_view() << "' : '"
              << sample.get_payload().as_string_view() << "')\n";
}

// non-blocking
for (bool call_success = recv(reply); !call_success || reply.check(); call_success = recv(reply)) {
  if (!call_success) {
    std::cout << ".";
    Sleep(1);
    continue;
  }
  auto sample = expect<Sample>(reply.get());
  std::cout << "\nReceived ('" << sample.get_keyexpr().as_string_view() << "' : '"
            << sample.get_payload().as_string_view() << "')";
}
```

In 1.0.0, `Subscriber`, `Queryable` and `get` can now use either a callable object or a stream handler. Currently, Zenoh provides 2 types of handlers:

- `FifoHandler` - serving messages in Fifo order, *blocking* once full.
- `RingHandler` - serving messages in Fifo order, *dropping older messages* once full to make room for new ones.

```cpp
// callback
session.get(
  keyexpr, "", on_reply, on_done,
  {.target = Z_QUERY_TARGET_ALL}
);

// stream handlers interface
auto replies = session.get(
  keyexpr, "", channels::FifoChannel(16), 
  {.target = QueryTarget::Z_QUERY_TARGET_ALL}
);
// blocking
for (auto res = replies.recv(); std::has_alternative<Reply>(res); res = replies.recv()) {
  const auto& sample = std::get<Reply>(res).get_ok();
  std::cout << "Received ('" << sample.get_keyexpr().as_string_view() << "' : '"
            << sample.get_payload().deserialize<std::string>() << "')\n";
}
// non-blocking
while (true) {
  auto res = replies.try_recv();
  if (std::has_alternative<Reply>(res)) {
    const auto& sample = std::get<Reply>(res).get_ok();
	  std::cout << "Received ('" << sample.get_keyexpr().as_string_view() << "' : '"
            << sample.get_payload().deserialize<std::string>() << "')\n";
  } else if (std::get<channels::RecvError>(res) == channels::RecvError::Z_NODATA) {
	  // try_recv is non-blocking call, so may fail to return a reply if the Fifo buffer is empty
	  std::cout << ".";
    std::this_thread::sleep_for(1s);
    continue;
  } else { // std::get<channels::RecvError>(res) == channels::RecvError::Z_DISCONNECTED
	  break; // no more replies will arrive
  }
}
std::cout << std::endl;
  
```

The same works for `Subscriber` and `Queryable`:

```cpp
// callback
auto data_callback = [](const Sample &sample) {
  std::cout << ">> [Subscriber] Received ('"
            << sample.get_keyexpr().as_string_view() 
            << "' : '" << sample.get_payload().deserialize<std::string>() 
            << "')\n";
};

auto subscriber = session.declare_subscriber(
  keyexpr, data_callback, closures::none // or dedicated function to call when subscriber is undeclared
);
std::cout << "Press CTRL-C to quit...\n";
while (true) {
  std::this_thread::sleep_for(1s);
}


// stream handlers interface
auto subscriber = session.declare_subscriber(keyexpr, channels::FifoChannel(16));
const auto& messages = subscriber.handler();
//blocking
for (auto res = messages.recv(); std::has_alternative<Sample>(res); res = messages.recv()) {
		// recv will block until there is at least one sample in the Fifo buffer
		// it will return an empty sample and alive=false once subscriber gets disconnected
		const Sample& sample = std::get<Sample>(res);
    std::cout << "Received ('" << sample.get_keyexpr().as_string_view() << "' : '"
              << sample.get_payload().deserialize<std::string>() << "')\n";
}
// non-blocking
while (true) {
  auto res = messages.try_recv();
  if (std::has_alternative<Sample>(res)) {
    const auto& sample = std::get<Sample>(res);
	  std::cout << "Received ('" << sample.get_keyexpr().as_string_view() << "' : '"
            << sample.get_payload().deserialize<std::string>() << "')\n";
  } else if (std::get<channels::RecvError>(res) == channels::RecvError::Z_NODATA) {
	  // try_recv is non-blocking call, so may fail to return a sample if the Fifo buffer is empty
	  std::cout << ".";
    std::this_thread::sleep_for(1s);
  } else { // std::get<channels::RecvError>(res) == channels::RecvError::Z_DISCONNECTED
	  break; // no more samples will arrive
  }
}
std::cout << std::endl;

```

## Attachment

In version 0.11.0 an attachment could only represent a set of key-value pairs and had a somewhat complicated interface:

```cpp
// publish message with attachment
options.set_encoding(Z_ENCODING_PREFIX_TEXT_PLAIN);
std::unordered_map<std::string, std::string> attachment_map = {
  {"source", "C++"},
  {"index", "0"}
};    
options.set_attachment(attachment_map);
pub.put(s, options);

// subscriber callback function receiving message with attachment
data_handler(const Sample &sample) {
  std::cout << ">> [Subscriber] Received " ('"
            << sample.get_keyexpr().as_string_view() 
            << "' : '" 
            << sample.get_payload().as_string_view()
		        << "')\n";
  if (sample.get_attachment().check()) {
      // reads full attachment
      sample.get_attachment().iterate([](const BytesView &key, const BytesView &value) -> bool {
          std::cout << "   attachment: " << key.as_string_view() << ": '" << value.as_string_view() << "'\n";
          return true;
      });

      // or read particular attachment item
      auto index = sample.get_attachment().get("index");
      if (index != "") {
          std::cout << "   message number: " << index.as_string_view() << std::endl;
      }
  }
};
```

In 1.0.0, attachment handling was greatly simplified. It is now represented as `Bytes` (i.e. the same class we use to represent serialized data) and can thus contain data in any format.


```c++
// publish a message with attachment
auto session = Session::open(std::move(config));
auto pub = session.declare_publisher(KeyExpr(keyexpr));
// send some key-value pairs as attachment
// allocate attachment map
std::unordered_map<std::string, std::string> attachment_map = {
  {"source", "C++"},
  {"index", "0"}
};    
pub.put(
  Bytes::serialize("my_payload"), 
  {.encoding = Encoding("text/plain"), .attachment = std::move(attachment_map)}
);


// subscriber callback function receiving a message with attachment
void data_handler(const Sample &sample) {
  std::cout << ">> [Subscriber] Received ('"
            << sample.get_keyexpr().as_string_view() 
            << "' : '" 
            << sample.get_payload().deserialize<std::string>()
            << "')\n";
  auto attachment = sample.get_attachment();
  if (!attachment.has_value()) return;
  // we expect attachment in the form of key-value pairs
  auto attachment_deserialized = attachment->get().deserialize<std::unordered_map<std::string, std::string>>();
  for (auto&& [key, value]: attachment) {
    std::cout << "   attachment: " << key << ": '" << value << "'\n";
  }
};
```


# Optional Parameters

Handling for optional parameters for Zenoh functions was simplified. There are no more getters/setters and all fields of option structures are public. Also option arguments are automatically set to their default values, and if your compiler has support for designated initializers, it is sufficient to only set the fields that are needed to be different from default ones.

In version 0.11.0:

```cpp
GetOptions opts;
opts.set_target(Z_QUERY_TARGET_ALL);
opts.set_value(value);

...

session.get(keyexpr, "", {on_reply, on_done}, opts);
```

In 1.0.0:

```cpp
session.get(keyexpr, "", on_reply, on_done, {.target = Z_QUERY_TARGET_ALL, .payload = Bytes::serialize(value)});
```