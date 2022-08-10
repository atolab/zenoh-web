---
title: "Your First zenoh app"
weight : 1000
menu:
  docs:
    parent: getting_started
---
Getting started with zenoh is quite straightforward. Below we will show you how to create a simple telemetry application. Let's assume that we have some sensor, say a temperature sensor, and we want to store this temperature into a zenoh storage. Later on, we want to retrieve this temperature from the zenoh storage. 

Before cranking some code, let's define some terminology. 

<b>Zenoh</b> deals with <i>keys/values</i> where each key is a <i>path</i> and is associated to a <i>value</i>. A path looks like just a Unix file system path, such as ```myhome/kitchen/temp```. The value can be defined with different
encodings (string, JSON, raw bytes buffer...). 

To query the values stored by zenoh, we use <i>selectors</i>. As the name suggest, a <i>selector</i> can use wildcards, such as <b>*</b> and <b>**</b> to represent a set of paths, such as, ```myhome/*/temp```.

Let's get started!

## Zenoh Programming in Python

By default, a zenoh router starts without any storage. In order to store the temperature, we need to configure one:  
create a `zenoh-myhome.json5` configuration file for zenoh with this content:
```json5
{
  plugins: {
    rest: {                        // activate and configure the REST plugin
      http_port: 8000              // with HTTP server listening on port 8000
    },
    storage_manager: {             // activate and configure the storage_manager plugin
      storages: {
        myhome: {                  // configure a "myhome" storage
          key_expr: "myhome/**",   // which subscribes and replies to query on /myhome/**
          volume: {                // and using the "memory" volume (always present by default)
            id: "memory"
          }
        }
      }
    }
  }
}
```

[Install](../installation) and start the zenoh router with this configuration file:

```bash
zenohd -c zenoh-myhome.json5
```


Now let's write an application that will produce temperature measurements at each second:

```python
import zenoh, random, time

random.seed()

def read_temp():
    return random.randint(15, 30)

def run_sensor_loop(session):
    # read and produce a temperature every second
    while True:
        t = read_temp()
        session.put('myhome/kitchen/temp', t)
        time.sleep(1)

if __name__ == "__main__":
    session = zenoh.open()
    run_sensor_loop(session)
```


Below is the application that will retrieve the latest temperature value stored in zenoh:

```python
import zenoh

if __name__ == "__main__":
    session = zenoh.open()
    results = session.get('myhome/kitchen/temp')
    key, value = results[0].data.key_expr, results[0].data.value.decode()
    print('  {} : {}'.format(key, value))
```

Finally, if ever we want to receive the temperatures in direct from the publisher,
without querying the zenoh storage, we can use a subscriber:

```python
import zenoh, time

def listener(sample):
    print('Publication received: {} = {}'
            .format(sample.key_expr, sample.value.decode()))

if __name__ == "__main__":
    session = zenoh.open()
    sub = session.subscribe('myhome/kitchen/temp', listener)
    time.sleep(60)
```

## Other code examples

Now you can also have a look to the examples provided with each client API:

 - **Rust**: https://github.com/eclipse-zenoh/zenoh/tree/master/examples
 - **Python**: https://github.com/eclipse-zenoh/zenoh-python/tree/master/examples
 - **C (zenoh-net)**: https://github.com/eclipse-zenoh/zenoh-c/tree/master/examples
