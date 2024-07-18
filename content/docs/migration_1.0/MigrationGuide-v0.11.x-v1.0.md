---
title: "Migrating from Zenoh v0.11.x to Zenoh v1.0"
weight : 6100
menu:
  docs:
    parent: migration_1.0
---

The engineers at ZettaScale have been hard at work preparing an official version 1.0.0 of Zenoh! 
This major release comes with several API changes, Quality of Life improvements and developer conveniences.

We now have a more stable API and intend to keep backward compatibility in the future Zenoh revisions.
This guide is here to ease the transition to Zenoh 1.0.0 for our users!

## Value is gone, long live ZBytes 
We have replaced Value with `ZBytes` and `Encoding`, and added a number of conversion implementations such that user defined structs can be serialized into ZBytes, sent via Zenoh, and de-serialized from `ZBytes` with ease.

Each Language bindings will have their own specifics of Serializing and Deserializing.


## Encoding
`Encoding` has been reworked. 
Zenoh does not impose any encoding requirement on the user, nor does it operate on it.

It can be thought of as optional metadata, carried over by Zenoh in such a way that the end user’s application may perform different operations based on encoding.

We have expanded our list of pre-defined encoding types from Zenoh 0.11.0 for user convenience. 

Users can also define their own encoding scheme that does not need to be based on the pre-defined variants.

## Attachment

## Query & Queryable


## Use accessors to get private members
Across language bindings we encapsulate members of structs, and they can’t be accessed directly now. 
The only way to access struct values is to use the getter function associated with them.


## Pull mode is no longer + Support RingChannel
We no longer support Pull mode in Zenoh.

Besides using a callback to receive data, we can also receive the data from a default FIFO channel. 

However, sometimes we only care about the latest data and want to discard the oldest data, in this case we support the use of a RingChannel un

# Storages
The storage-manager will fail to launch if the timestamping configuration option is disabled.

From Zenoh 1.0.0 user-applications can load plugins.

A, somehow, implicit assumption that dictated the behaviour of storages is that the Zenoh node loading them **has to add a timestamp to any received publication that did not have one**. This functionality is controlled by the `timestamping` configuration option.

Until Zenoh 1.0.0 this assumption held true as only a router could load storage and the default configuration for a router enables `timestamping`. However, in Zenoh 1.0.0 nodes configured in `client` & `peer` mode can load storage and *their default configuration disables `timestamping`*.