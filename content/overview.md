---
title: "What is Zenoh?"
menu: overview
---

*(Motivate why Zenoh is needed, problem statement)*

Zenoh is the Next Big Thing in Internet Computing. You may think this is a bold statement, but hopefully after this short read you'll share the perspective.

One way of thinking about Zenoh is to  imagine it as a data liberator protocol. Zenoh liberates data in several dimensions.

**Cloud to the Microcontroller Communication.** Zenoh is the only protocol available on the market that can work efficiently and perform from server-grade hardware and networks to the embedded microcontroller and extremely constrained networks. As a consequence, Zenoh liberates the data allowing it to freely flow vertically and horizontally from the microcontroller to the data-center. Likewise, it liberates developers from the need to integrate technologies to bridge the communication between the enterprise and the embedded world..
  
**Data Centricity and Location Transparency for Data in Movement and at Rest.**  Since the introduction of Publish/Subscribe the world of data in movement has enjoyed location transparency. In other terms, the ability of receiving data without having to care about the location of the publisher.  Location transparency is a consequence of data centricity -- in these systems users only need to express interests without any concern on the location of its source. This feature is extremely important as it makes it easier to deal with scale, failures, and load-balancing. Zenoh is the first technology to bring location transparency for data at rest, allowing queries to be expressed without any concerns on the actual location of data, i.e. the location of the data-bases. It is Zenoh that takes care of identifying the optimal set of data bases available in  the network where the query should  be executed.

**Easy to Use and Performant.** Zenoh has been designed ground up to be simple to use and high-performance across all its applicability range. It constantly delivers higher throughput and lower latency and achieves that in a fraction of the code of competing protocols. This ensures that developers can be productive from day one and don't need to write brittle and unmaintainable code in order to get performance.

**Energy Efficient.** Few people are aware that communication is incredibly energivore. Think about the difference in the battery life of your mobile when surfing the internet. Now realize that the bulk of the energy that is being used is in the networks that connect you to the data-centers. Zenoh was designed for energy efficiency. This is reflected in its minimal wire-overhead of 4-6 bytes, and its support for exploiting locality in communication.

**Adopted by Next Generation Applications.** In spite of its young age, Zenoh has witnessed an incredibly swift adoption and is used today in next generation applications, such as, Robotics, Autonomous Vehicles, Internet Gaming, and Telecommunications. Specifically, in Robotics, Zenoh is emerging as the protocol of choice for Robot-to-Robot communication and  Internet-scale monitoring management and real-time teleoperation. Likewise, several Autonomous Vehicles and Mobility initiatives have adopted Zenoh for Vehicle-to-Anything communication and in a growing number of cases as the only protocol used for on- and off- vehicle communication.

Zenoh has achieved all of this by extremely careful design and craftsmanship. It is the first protocol available on the market that has managed to integrate Internet-Scale Publish/ Subscribe with Geo-Distributed Queries. 

Let us now look into a sample scenario of Zenoh working.
Zenoh supports two paradigms of communication - publish-subscribe and queries. 
<!-- The source is available at https://drive.google.com/file/d/1exhGofWDyiEIES_WICsslXWZ3yiTSkAl/view?usp=sharing (ATO/Techno/Slides/zenoh/2022/2022.09.08-zenoh-web-animation.key) 
Settings::: Resolution: Extra Large, Frame Rate 30 fps, Export with transparent backgrounds -->

- [Pub/Sub in Zenoh](#pubsub-in-zenoh)
- [Query in Zenoh](#queries-in-zenoh)

## Pub/Sub in Zenoh
![Zenoh pub/sub in action](/img/zenoh-pub-sub.gif "Zenoh pub/sub in action")

This animation shows a basic pub/sub in action. The subscribers connected to the system receives the values send by the publishers routed efficicently through the Zenoh network.
You can also observe the presence of a sleeping subscriber connected to the network. Once the subscriber awakes, the nearest Zenoh node will send the pending publications.

## Queries in Zenoh
![Zenoh queries in action](/img/zenoh-query.gif "Zenoh queries in action")

This animation shows a simple query in action through Zenoh. You can see the presence of storages and queryables. 
A queryable is any process that can reply to queries. A storage is a combination of a subscriber and a queryable.

--

<!-- In the following sections, we introduce the primitives and abstractions in Zenoh that enables this sample scenario. -->
Seeing all the wonderful things that comes with Zenoh, why not [get started](../docs/getting-started/first-app) with it now?  