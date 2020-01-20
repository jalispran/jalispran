---
title: "Zookeeper - All you need to know"
layout: post
date: 2020-01-20
tag: 
    - Zookeeper
    - Distributed Systems
    - Coordination Service
    - Znodes
    - Quorum
description: "This blog covers the basic of Zookeeper"
image: https://upload.wikimedia.org/wikipedia/en/thumb/8/81/Apache_ZooKeeper_Logo.svg/1024px-Apache_ZooKeeper_Logo.svg.png
headerImage: https://upload.wikimedia.org/wikipedia/en/thumb/8/81/Apache_ZooKeeper_Logo.svg/1024px-Apache_ZooKeeper_Logo.svg.png
category: blog
author: pranjal
---

Historically, each application was a single program running on a single computer with a single CPU. Today, things have changed. In the Big Data and Cloud Computing world, applications are made up of many indepndant programs running on an ever-changing set of computers.

Coordinating the actions of these independant programs is far more difficult than writing a single program to run on a single computer.

**Zookeeper** was designed to be a robust service that enables application developers to mainly focus on their application logic rather than coordination. It exposes a simple API, inspired by the filesystem API, that allows developers to implement common coordination tasks.

### Where is Zookeeper used?
- Apache HBase
- Apache Kafka
- Apache Solr
- Yahoo! Fetching Services
- Facebook Messenger 

## CAP Theorem
It is impossible for a dostributed data store to simultaneously provide more than two out of the following three gurantees.
- _Consistency_: Every read receives the most recent write or an error
- _Availability_: Every request receives a (non-error) response - without the gurantee that it contains the most recent write
- _Partition tolerance_: the system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes

It implies that in the presence of a network partition, one has to choose between _Consistency_ and _Availability_

Zookeeper is designed with mostly Consistency and Availability in mind. It also provides read-only capability in the presence of network partition. What that means is, the clients connected to a znode, which was part of an ensamble/cluster, can keep communicating with the znode (even after network partition) and will keep receiving stale, read-only data.

_Paxos_ and _Virtual Synchrony_ algorithms have been perticularly influential in the design of Zookeeper

> Zookeeper is a CP system

## Definitions
#### znode
Every node in a Zookeeper tree is referred to as a _znode_

#### watch
A watch is a one-shot operation, which means that it triggers one notification 

#### quorum
Zookeeper quorum is the minimum number of servers that have to be running and available in order for Zookeeper to work

#### zxid
A _zxid_ is a long (64 bit) integer split into two parts: the  _epoch_ and the _counter_. Each part has 32 bits. Simply put, _zxid_ is the timestamp of the last change

#### leader
The _leader_ is the central point for handling all requests that change the Zookeeper system. It acts as a sequencer and establishes the order of updates to the Zookeeper state.

#### follower
A _follower_ receive and vote on the updates proposed by the leader to guarantee that updates to the state survive crashes

#### split-brain
Two subsets of servers making progress simultaneously. In this scenario, a single cluster can have multiple Leaders. Zookeeper strongly advises **against** _split-brain_. Needless to say, a _split-brain_ points to a faulty configuration

> Myth-buster: Zookeeper is not for bulk storage

## znode in detail
#### Persistent znode
A persistent znode _/path_ can be deleted only through a call to _delete/remove_ (there is a slight difference between the two)

By default, all znodes are persistent znodes unless otherwise stated

#### Ephemeral znode
An _ephemeral znode_ is deleted if the client that created it crashes or simply closes the connection to Zookeeper.

#### Sequential znode
A _sequential znode_ is assigned a unique, monotonically increasing integer

There are four options for the mode of a znode:
`persistent, ephemeral, persistent_sequential and ephemeral_sequential `

## watch in detail
To replace client polling, Zookeeper has opted for a mechanism based on notifications. Clients register with Zookeeper to receive notifications of changes ro znodes. Registering to receive a notification for a given znode consists of setting a _watch_.

_Watches_ are one time triggers and are sent asynchronously to the watchers/client. 

_Watches_ are set while reading data and triggered while writing data.

> Notifications are delivered to a client before any other change is made to the same node

_Watches_ only tell that something has changed, it does not talk about what has changed.

A client can set a _watch_ for:
- changes to the data of a znode
- changes to the children of a znode
- znode being created or deleted

## quorum in detail
This number is the minimum number of servers that have to store client's data before telling the client that its data is safely stored.

We should always shoot for an odd number of servers. Typically, `(2F + 1)` servers. Where `F` is the number of server failures the cluster can tolerate

## Versions
A couple of operations in the Zookeeper API can be executed conditionally: `setData` and `delete`. Both calls take a _version_ as an input parameter and the operation succeeds only if the _version_ passed by the client matches the current _version_ on the server.

The use of _version_ is important when multiple clients might be trying to perform operations over the same znode.

## Sessions
Before executing any request against a Zookeeper ensemble, a client must establish a session with the Zookeeper service.

As soon as a client is connected to a server and a session established, the session is replicated with the leader.

Sessions offer order grantees, which means, requests in a session are executed in `FIFO` order. However, a client can have multiple concurrent sessions, in which case `FIFO` ordering is not preserved across sessions.

Sessions may be moved to a different server of the client has not heard from its current server for some time. 
> Moving the session to a different server, transparently, is handled by the client library

All operations a client submits to Zookeeper are associated to a session. When a session ends for any reason, the ephemeral nodes created dusring that session disappear.

The Zookeeper ensamble is the one responsible for declaring session expired, not the client. The client may choose to close the session, however.

#### Session handling on the client side
On the client side, if it has heard nothing from the server at the end of `1/3rd of t`, it sends a _heartbeat_ message to the server. At `2/3rd of t` the client starts looking for another server, and it has another `1/3rd of t` to find one.

## Stay tuned for season 2
There is so much more to Zookeeper than this. Lets not overwhelm ourselves with information. Take time to process all this and in the meanwhile, allow me to write another blog in the continuation.

I will cover the following topics in next post:
- Types of servers
- Leader election
- How to configure Zookeeper in standalone mode and in ensemble mode
- Notification messages
- Logs

See you soon! 

 