---
title: "What is zenoh?"
weight: 1
menu: "docs"
---

With the steady increase in the number of network connected devices we are experiencing a new level of heterogeneity with respect to computing, storage and communication capabilities, as well as new challenges with respect to the scale at which data is produced and needs to be consumed. 

Additionally, for performance, efficiency and privacy reasons, there is an increasing desire to keep data processing as close as possible to the source, while at the same time not hindering access to geographically remote applications. In other terms, we are experiencing a mainstream [architectural switch](https://perspectives.tech/2019/12/10/architectural-liberum-arbitrium/) from cloud-centric paradigms in which the cloud stores, processes and retrieves data to fog and [edge-centric](https://edgenative.eclipse.org/) paradigms which store and process data where it makes most sense for performance, energy efficiency and security matters. 

**zenoh** has been designed to address the needs of those applications that deal with data in movement, data at rest and computation in a scalable, efficient and location transparent data manner.  

**zenoh** unifies data in motion, data in-use, data at rest and computations. It carefully blends traditional pub/sub with geo-distributed storages, queries and computations, while it retains a level of time and space efficiency that is well beyond any of the mainstream stacks. 

**zenoh** is designed to:

- Provide a small set of primitives to deal with data in motion, data at rest and computations.

- Give total control on storage location and back-end technology integration.

- Minimize network overhead – the minimal wire overhead of a data message is 4 bytes.

- Support extremely constrained devices – its footprint on Arduino Uno is 300 bytes.

- Support devices with low duty-cycle as it allows the negotiation of data exchange modes and schedules.
  
- Provide a rich set of abstraction to distribute, query and store data along the entire system. 
  
- Provide extremely low latency and high throughput. We also provide analytical and empirical comparison of zenoh’s efficiency against mainstream protocols such as DDS and MQTT.

- Make a lower-level API available, namely **zenoh-net**, which gives you full control on zenoh primitives.