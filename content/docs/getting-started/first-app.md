---
title: "Your First zenoh App"
weight : 1020
menu:
  docs:
    parent: getting_started
---
Getting started with zenoh is quite straightforward. Below we will show you how to create a simple telemetry application. Let's assume that we have some sensor, say a temperature sensor, and we want to make this temperature available to interested applications. 

We will do the same example in multiple programming languages. Before cranking some code, let's define some terminology. 

<b>zenoh</b> deals with <i>resources</i> where a resource has a <i>path</i> and a <i>value</i>. A path looks like just a Unix file system path, such as ```/home-id/kitchen/temp```. For what concerns the value, while zenoh allows you to associate an encoding to your data, it treats data as raw bytes. 

Almost all primitives provided by zenoh operate on <i>selectors</i>. As the name suggest, a <i>selector</i> can uses wildcards, such as <b>*</b> and <b>**</b> to represent a set of paths, such as, ```/home-id/*/temp```.

Let's get started!

# zenoh Programming in Python 
Let's write first an application that will produce temperature measurements.

```python
from zenoh import Zenoh 
import random
import time

random.seed()

def read_temp():
    return random.randint(15, 30)    

def run_sensor_loop(z, pub):
    # read and produce e temperature every second
    while True:
        t = read_temp()
        z.stream_data(pub, str(t).encode())
        time.sleep(1)

if __name__ == "__main__":        
    z = Zenoh.open(None)
    pub = z.declare_publisher('/myhome/kitcken/temp')
    run_sensor_loop(z, pub)
```


Below is the application that will subscribe to this resource for ten seconds and then clean-up and exit.


```python
from zenoh import Zenoh, SubscriberMode
import time

def listener(rname, data, info):
    print("{}: {}".format(rname, data.decode()))
    

if __name__ == "__main__":        
    z = Zenoh.open()
    sub = z.declare_subscriber('/myhome/kitcken/temp', SubscriberMode.push(), listener)
    
    # Listen for one minute and then exit
    time.sleep(10)
    z.undeclare_subscriber(sub)
    z.close()
```


