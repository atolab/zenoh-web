---
title: "Python"
weight : 6500
menu:
  docs:
    parent: migration_1.0
---

## Highlights

The library has been fully rewritten to use only Rust. It should make no difference for users, except you may observe a significant performance improvement, up to x5.

The API has also been reworked to feel more pythonic, using notably context managers.

## Context managers and background execution

You *should* close the zenoh session after use and the recommended way is through context manager:

```python
import zenoh
with zenoh.open(zenoh.Config()) as session:
    # `session.close()` will be called at the end of the block
```

Session-managed objects like subscribers or queryables can also be managed using context managers:

```python
with session.declare_subscriber("my/keyexpr") as subscriber:
    # `subscriber.undeclare()` will be called at the end of the block`
```

However, these objects can also be used without context manager, and without calling `undeclare`. In that case, they will run in “background” mode, meaning that their lifetime will be bound on the session’s one.

```python
import zenoh
with zenoh.open(zenoh.Config()) as session:
    subscriber = session.declare_subscriber("my/keyepxr")
    for sample in subscriber:
        ...
    # `session.close()` will be called at the end of the block,    # and it will undeclare the subscriber
```

## ZBytes, encoding, and (de)serialization

### Encoding

`zenoh.Value` has been split in `zenoh.ZBytes` and `zenoh.Encoding`. Put and other operations now requires a `ZBytes` payload, and accept an optional `Encoding`; there is no more auto-encoding from the payload type.

```python
session.put("my/keyexpr", 42) # default encoding `zenoh/bytes`session.put("my/keyexpr", 42, encoding=zenoh.Encoding.ZENOH_INT64)
```

Publishers can be declared with a default encoding, which will be used for each put operations.

```python
import json
publisher = session.declare_publisher("my/keyepxr", encoding=zenoh.Encoding.APPLICATION_JSON)
publisher.put(json.dumps({"key", "value"}))  # default encoding from publisher `application/json`
```

### (De)serialization

Arbitrary types can be serialized to and deserialized from `ZBytes`. Default (de)serializers are provided for builtin types; `list`/`dict` are **no more** serialized to JSON, they use instead zenoh builtin serializer, compatible with other zenoh bindings.

```python
payload = zenoh.ZBytes(42)
assert payload.deserialize(int) == 42# `ZBytes.deserialize` accepts generic `list`/`dict` typepayload = zenoh.ZBytes([0.5, 37.1])
assert payload.deserialize(list[float]) == [0.5, 37.1]
```

(De)serializers can be registered for custom types

```python
from dataclasses import dataclass
import zenoh
@dataclassclass RGB:
    red: int    green: int    blue: int@zenoh.serializerdef serialize_rgb(rgb: RGB) -> zenoh.ZBytes:
    return zenoh.ZBytes(rgb.red | (rgb.green << 8) | (rgb.blue << 16))
@zenoh.deserializerdef deserialize_rgb(payload: zenoh.ZBytes) -> RGB:
    compact = payload.deserialize(int)
    return RGB(compact & 255, (compact >> 8) & 255, (compact >> 16) & 255)
color = RGB(61, 67, 97)
assert zenoh.ZBytes(color).deserialize(RGB) == color
# types with a registered serializer can be used directly with `put`session.put("my/keyexpr", color)
```

## Handlers

The library now directly exposes Rust-backed handlers in `zenoh.handlers`. When no handler is provided, `zenoh.handlers.DefaultHandler` is used.

```python
import zenoh.handlers
subscriber = session.declare_subscriber("my/keyexpr", zenoh.handlers.DefaultHandler())
# equivalent to `session.declare_subscriber("my/keyexpr")`# builtin handlers provides `try_recv`/`recv` methods and can be iterated sample_or_none = subscriber.handler.try_recv()
sample = subscriber.handler.recv()
for sample in subscriber.handler:
    ...
# builtin handlers methods can be accessed directly through subscriber/queryable object sample_or_none = subscriber.try_recv()
sample = subscriber.recv()
for sample in subscriber:
    ...
```

Callbacks can also be used as handler:

```python
def handle_sample(sample: zenoh.Sample):
    ...
session.declare_subscriber("my/keyexpr", handle_sample)
# A drop callback can be passed using `zenoh.handlers.Callback`def stop():
    ...
session.declare_subscriber("my/keyexpr", zenoh.handlers.Callback(handle_sample, stop))
```

Note that for each callback handler, zenoh will in fact use a builtin handler and spawn a thread iterating the handler and calling the callback. This is needed to avoid GIL-related issues in low-level parts of zenoh, and as a result, leads to a significant performance improvement.