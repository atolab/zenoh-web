---
title: "Your first zenoh app"
weight : 1020
menu:
  docs:
    parent: getting_started
---
Getting started with zenoh is quite straightforward. This page shows you how to create a simple telemetry application. In this example, we have a temperature sensor, we want to store this temperature into zenoh storage and retrieve it from the zenoh storage. 

<b>zenoh</b> deals with <i>keys/values</i> where each key is a <i>path</i> and is associated with a <i>value</i>. A path looks like a Unix file system path, such as ```/myhome/kitchen/temp```. The value can be defined with different encodings (string, JSON, raw bytes buffer...). 

To query the values stored by zenoh, we use <i>selectors</i>. A <i>selector</i> can use wildcards, such as <b>*</b> and <b>**</b> to represent a set of paths, for example, ```/myhome/*/temp```.

## zenoh programming in Python 

By default, a zenoh router starts without any storage. In order to store the temperature, we need to add one, we must start `zenohd` with the `--mem-storage` option. The following command starts the router with a storage in zenoh memory that stores any key starting with `/myhome/`:

```bash
zenohd --mem-storage='/myhome/**'
```


The following writes an application that produces temperature measurements at each second:

```python
from zenoh import Zenoh
import random
import time

random.seed()

def read_temp():
    return random.randint(15, 30)

def run_sensor_loop(w):
    # read and produce a temperature every second
    while True:
        t = read_temp()
        w.put('/myhome/kitchen/temp', t)
        time.sleep(1)

if __name__ == "__main__":
    z = Zenoh({})
    w = z.workspace('/')
    run_sensor_loop(w)
```


The following is an application to retrieve the latest temperature value stored in zenoh:

```python
from zenoh import Zenoh

if __name__ == "__main__":
    z = Zenoh({})
    w = z.workspace('/')
    results = w.get('/myhome/kitchen/temp')
    key, value = results[0].path, results[0].value
    print('  {} : {}'.format(key, value))
```


To receive the temperatures direct from the publisher, without querying the zenoh storage, we can use a subscriber:

```python
from zenoh import Zenoh, ChangeKind
import time

def listener(change):
    if change.kind == ChangeKind.PUT:
        print('Publication received: "{}" = "{}"'
                .format(change.path, change.value))

if __name__ == "__main__":
    z = Zenoh({})
    w = z.workspace('/')
    results = w.subscribe('/myhome/kitchen/temp', listener)
    time.sleep(60)
```

## Other code examples

The following examples are provided for each client API:

 - **Rust**: https://github.com/eclipse-zenoh/zenoh/tree/master/zenoh/examples/zenoh
 - **Rust (zenoh-net)**: https://github.com/eclipse-zenoh/zenoh/tree/master/zenoh/examples/zenoh-net
 - **C (zenoh-net)**: https://github.com/eclipse-zenoh/zenoh-c/tree/master/examples/net
 - **Python**: https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh
 - **Python (zenoh-net)**: https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh-net