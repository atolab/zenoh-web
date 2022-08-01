---
title: "Key concepts"
weight : 1000
menu:
  docs:
    parent: getting_started
---

## Deployment units

zenoh provides 3 kinds of deployment units: **peers**, **clients** and **routers**.

### peer application
A user application using a zenoh API and able to:
<div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">
            Communicate with other <b>peers</b> over a 
            <a href="https://en.wikipedia.org/wiki/Complete_graph">complete graph</a> topology.
        </div>
        <div style="height:20px;text-align:center;"></div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="/img/peer_to_peer.png" alt="peer to peer" width="150"></img>
        </div>
    </div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">
            Communicate with other <b>peers</b> over <a href="https://en.wikipedia.org/wiki/Connectivity_(graph_theory)#Connected_vertices_and_graphs">connected graph</a>  topology.</div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="/img/peers_mesh.png" alt="peers mesh" width="150"></img>
        </div>
    </div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">
            Communicate across the Internet through <b>zenoh routers</b>.
        </div>
        <div style="height:20px;text-align:center;"></div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="/img/routed_peers.png" alt="routed peers" width="150"></img>
        </div>
    </div>
</div>
<br style="clear:both;"></br>

### client application
A user application using a zenoh API and that connects to a single **zenoh router** (or a single **peer**) to communicate with the rest of the system.

<div style="height:200px;display:flex;justify-content: center;align-items: center;">
    <img src="/img/routed_clients.png" alt="routed clients" width="200"></img>
</div>

### zenoh router
A software process able to route the zenoh protocol between **clients** and **peers** in any given topology.

<div style="display:flex;justify-content: center;align-items: center;">
    <img src="/img/full_topology.png" alt="full topology" width="600"></img>
</div>

The router executable file is named `zenohd` and is available in [zenoh releases](./installation#installing-zenohs-router), as a [Docker image](./quick-test) or [building it by yourself](https://github.com/eclipse-zenoh/zenoh#how-to-build-it).

------
## User API

The zenoh API provides the primitives to allow pub/sub (push) communications as well as query/reply (pull) communications:
 - **put :** push live data to the matching subscribers and storages.
 - **subscribe :** subscriber to live data publications.
 - **get :** query data from the matching storages and evals.
 - **queryable :** declares an entity able to reply to queries

Based on those primitives, zenoh introduces the **Storage** abstraction that uses both **subscribe** (to receive and store publications) and **queryable** (to reply to queries with stored data).

![key primitives](/img/key_primitives_v0.6.png "key primitives")

The zenoh API is provided for a set of programming languages (full list in [API documentations](../APIs/APIs)). Each one implements the zenoh protocol to support those primitives.  
Thanks to limited prerequisites, the zenoh protocol can be implemented on top of either Physical, Data Link or Transport [communication layers](https://en.wikipedia.org/wiki/OSI_model).