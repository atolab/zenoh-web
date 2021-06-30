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
{{< rawhtml >}}
<div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">Communicate with other <b>peers</b> in a <i>peer-to-peer</i> topology</div>
        <div style="height:20px;text-align:center;""></div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="../../../img/peer_to_peer.png" alt="peer to peer" width="150"></img>
        </div>
    </div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">Communicate with other <b>peers</b> in a <i>mesh</i> topology</div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="../../../img/peers_mesh.png" alt="peers mesh" width="150"></img>
        </div>
    </div>
    <div style="float:left;width:33%;padding:20px;">
        <div style="height:50px;">Communicate with a wide system through <b>zenoh routers</b></div>
        <div style="height:20px;text-align:center;""></div>
        <div style="height:250px;display:flex;justify-content:center;align-items:center;">
            <img src="../../../img/routed_peers.png" alt="routed peers" width="150"></img>
        </div>
    </div>
</div>
<br style="clear:both;"></br>
{{< /rawhtml >}}

### client application
A user application using a zenoh API and that connects to a single **zenoh router** (or a single **peer**) to communicate with the rest of the system.
{{< rawhtml >}}
    <div style="height:200px;display:flex;justify-content: center;align-items: center;">
        <img src="../../../img/routed_clients.png" alt="routed clients" width="200"></img>
    </div>
{{< /rawhtml >}}

### zenoh router
A software process able to route the zenoh protocol between **clients** and **peers** in any given topology.
{{< rawhtml >}}
    <div style="display:flex;justify-content: center;align-items: center;">
        <img src="../../../img/full_topology.png" alt="full topology" width="600"></img>
    </div>
{{< /rawhtml >}}  
The router executable file is named `zenohd` and is available in [zenoh releases](../installation/#installing-zenohs-router), as a [Docker image](../quick-test) or [building it by yourself](https://github.com/eclipse-zenoh/zenoh#how-to-build-it).

------
## User APIs

zenoh provides 2 levels of API:

### zenoh-net
A network oriented API providing the key primitives to allow pub/sub (push) communications as well as query/reply (pull) communications. The zenoh-net layer only cares about data transportation and doesn't care about data content nor storing data.

### zenoh
A higher level API providing the same abstractions as the zenoh-net API in a simpler and more data-centric oriented manner as well as providing all the building blocks to create a distributed storage. The zenoh layer is aware of the data content and can apply content-based filtering and transcoding. 

  ![key primitives](../../../img/key_primitives.png "key primitives")

### zenoh-net primitives
 - **write :** push live data to the matching subscribers.
 - **subscribe :** subscriber to live data. 
 - **query :** query data from the matching queryables.
 - **queryable :** an entity able to reply to queries.

### zenoh primitives
 - **put :** push live data to the matching subscribers and storages. (equivalent of zenoh-net write)
 - **subscribe :** subscriber to live data.  (equivalent of zenoh-net subscribe)
 - **get :** get data from the matching storages and evals.  (equivalent of zenoh-net query)
 - **storage :** the combination of a zenoh-net subscriber to listen for live data to store and a zenoh-net queryable to reply to matching get requests.
 - **eval :** an entity able to reply to get requests. Typically used to provide data on demand or build a RPC system.  (equivalent of zenoh-net queryable)