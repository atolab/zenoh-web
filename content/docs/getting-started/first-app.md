---
title: "Your First zenoh app"
weight : 1020
menu:
  docs:
    parent: getting_started
---
Getting started with zenoh is quite straightforward. Below we will show you how to create a simple telemetry application. Let's assume that we have some sensor, say a temperature sensor, and we want to store this temperature into a zenoh storage. Later on, we want to retrieve this temperature from the zenoh storage. 

Before cranking some code, let's define some terminology. 

<b>zenoh</b> deals with <i>keys/values</i> where each key is a <i>path</i> and is associated to a <i>value</i>. A path looks like just a Unix file system path, such as ```/myhome/kitchen/temp```. The value can be defined with different
encodings (string, JSON, raw bytes buffer...). 

To query the values stored by zenoh, we use <i>selectors</i>. As the name suggest, a <i>selector</i> can uses wildcards, such as <b>*</b> and <b>**</b> to represent a set of paths, such as, ```/myhome/*/temp```.

Let's get started!

## zenoh Programming in Python 

By default, a zenoh router starts without any storage. In order to store the temperature, we need to add one,
starting `zenohd` with the `--mem-storage`option.
The following command starts the router with a storage in zenoh memory that will store any key starting with `/myhome/`:

```bash
zenohd --mem-storage='/myhome/**'
```


Now let's write an application that will produce temperature measurements at each second:

```python
from zenoh import Zenoh
import random
import time

random.seed()

def read_temp():
    return random.randint(15, 30)

def run_sensor_loop(w):
    # read and produce e temperature every second
    while True:
        t = read_temp()
        w.put('/myhome/kitcken/temp', t)
        time.sleep(1)

if __name__ == "__main__":
    z = Zenoh({})
    w = z.workspace('/')
    run_sensor_loop(w)
```


Below is the application that will retrieve the latest temperature value stored in zenoh:

```python
from zenoh import Zenoh

if __name__ == "__main__":
    z = Zenoh({})
    w = z.workspace('/')
    results = w.get('/myhome/kitcken/temp')
    key, value = results[0].path, results[0].value
    print('  {} : {}'.format(key, value))
```


Finally, if ever we want to receive the temperatures in direct from the publisher,
without querying the zenoh storage, we can use a subscriber:

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
    results = w.subscribe('/myhome/kitcken/temp', listener)
    time.sleep(60)
```

## Other code examples

Now you can also have a look to the examples provided with each client API:

 - **Python**: https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples/zenoh
 - **C**:   https://github.com/eclipse-zenoh/zenoh-java/tree/master/examples/net
