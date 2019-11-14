---
title: "What is zenoh?"
weight: 1
menu: "docs"
---

As a consequence of the widespread adoption of Cyber Physical Systems (CPS), the number of network connected devices is steadily increasingly as is (1) their heterogeneity with respect to computing, storage and communication capabilities and (2) the scale at which they produce and consume data. 

Thus, data exchange protocols face new needs with respect to vertical and horizontal scalability, support for constrained networks and devices with low duty cycle – in other terms devices that are disconnected/sleeping most of the time. 

Protocols used today to build these systems, such as MQTT, DDS, CoAP and HTTP were not designed with these needs in mind. As a result, architects and developers are forced into patchwork design in which multiple protocols are stitched together to provide some meaningful end-to-end semantics. 

Over the past decades, our team co-invented and built some of the communication infrastructures deployed today as part of telecommunication, aerospace and early Industrial Internet applications. It is with this baggage of experience that we came to the realization that a new protocol was needed – a protocol designed ground-up to address the needs of large scale CPS end-to-end. 

<b>zenoh</b>  is the results of this reflections and it is today the only protocol we know of that provides unified abstractions for dealing with data in motion as well as data at rest with zero overhead. 

<b>zenoh</b> has been designed to:
<ul> 
    <li>
        Minimize network overhead – the minimal wire overhead of a data message a data message is 4 bytes.
    </li>
    <li>
        Support extremely constrained devices – its footprint on Arduino Uno is of 300 bytes.
    </li>
    <li>
        Supports devices with low duty-cycle by allowing the negotiation of data exchange modes and schedules.
    </li>
    <li>
        Provides a rich set of abstraction for distributing, querying and storing data along the entire system. 
    </li>
    <li> 
        Provide extremely low latency and high throughput. We also provide analytical and empirical comparison of zenoh ’s efficiency against mainstream protocols such as DDS and MQTT.
    </li>
<ul>
