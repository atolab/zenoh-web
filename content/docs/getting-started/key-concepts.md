---
title: "Key concepts"
weight : 1005
menu:
  docs:
    parent: getting_started
---

## Deployment units

Zenoh provides 3 kinds of deployment units: **peers**, **clients** and **routers**.

### Zenoh peer
A user application using a Zenoh API and able to:
- Communicate with other **peers** over a [complete graph](https://en.wikipedia.org/wiki/Complete_graph) topology.
    > ![peer to peer](/img/peer_to_peer.png "peer to peer")

- Communicate with other **peers** over [connected graph](https://en.wikipedia.org/wiki/Connectivity_(graph_theory)#Connected_vertices_and_graphs) topology.
    > ![peers mesh](/img/peers_mesh.png "peers mesh")

- Communicate across the Internet through **Zenoh routers**.
    > ![routed peers](/img/routed_peers.png "routed peers")

### Zenoh client
A user application using a Zenoh API and that connects to a single **Zenoh router** (or a single **peer**) to communicate with the rest of the system.

![routed clients](/img/routed_clients.png "routed clients")

### Zenoh router
A software process able to route the Zenoh protocol between **clients** and **peers** in any given topology.

![full topology](/img/full_topology.png "full topology" )

The router executable file is named `zenohd` and is available in [Zenoh releases](../installation#installing-the-zenoh-router), as a [Docker image](../quick-test) or [building it by yourself](https://github.com/eclipse-zenoh/zenoh#how-to-build-it).

------
## User API

Zenoh API provides the primitive entities and operations essential for pub/sub and query/reply communications:
 - **publisher :** an entity that publishes live data
 - **subscriber :** an entity that subcribes to live data from the publisher
 - **queryable :** an entity that can reply to queries 
 - **put :** operation to put data
 - **get :** operation to fetch data

<!-- The Zenoh API provides the primitives to allow pub/sub communications as well as query/reply communications:
 - **put :** push live data to the matching subscribers and storages.
 - **subscribe :** subscribe to live data publications (includes an option to do a pull subscription).
 - **get :** query data from the matching queryables (including storages).
 - **queryable :** declare an entity able to reply to queries. -->

Based on these primitives, Zenoh introduces the **Storage** abstraction that is both a **subscriber** (to receive and store publications) and a **queryable** (to reply to queries with stored data).

![key primitives](/img/key_primitives_v0.6.png "key primitives")

The Zenoh API is provided for a set of programming languages (full list in [API documentations](../APIs/APIs)). Each one implements the Zenoh protocol to support those primitives.  
<!-- ~~Thanks to limited prerequisites, the Zenoh protocol can be implemented on top of either Physical, Data Link or Transport [communication layers](https://en.wikipedia.org/wiki/OSI_model).~~ -->