---
title: "What is zenoh?"
weight: 1
menu: "docs"
---

With the steady increase in the number of network connected devices we are experiencing a new level of heterogeneity with respect to computing, storage and communication capabilities, as well as new challenges with respect to the scale at which data is produced and needs to be consumed. 

Additionally, for performance, efficiency and privacy reasons, there is an increasing desire to keep the data processing as close as possible to the source, while at the same time not hindering access to geographically remote applications. In other terms, we are experiencing a mainstream [architectural switch](https://perspectives.tech/2019/12/10/architectural-liberum-arbitrium/) from cloud-centric paradigms in which data is stored, processed and retrieved from the cloud to fog and [edge-centric](https://edgenative.eclipse.org/) paradigms where data is stored and processed where it makes most sense for performance, energy efficiency and security matters. 

~~**Zenoh** has been designed to address the needs of those applications that need to deal with data in movement, data at rest and computation in a scalable, efficient and location transparent manner.~~

**Zenoh** unifies data in motion, data in use, data at rest and computations.  It carefully blends traditional pub/sub with geo-distributed storages, queries and computations, while retaining a level of time and space efficiency that is well beyond any of the mainstream stacks. 


**Zenoh** has been designed to:

- Provide a small set of primitives to deal with data in motion, data at rest and computations.

- Give total control on storage location and back-end technology integration.

- Minimize network overhead – the minimal wire overhead of a data message is 4 bytes.

- Support extremely constrained devices – its footprint on Arduino Uno is of 300 bytes.

- Supports devices with low duty-cycle by allowing the negotiation of data exchange modes and schedules.
  
- Provides a rich set of abstraction for distributing, querying and storing data along the entire system. 
  
- Provide extremely low latency and high throughput. ~~We also provide analytical and empirical comparison of zenoh’s efficiency against mainstream protocols such as DDS and MQTT.~~ 
<!-- This last sentence feels out of place -->
    