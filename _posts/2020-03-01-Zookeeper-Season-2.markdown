---
title: "Zookeeper - Season 2"
layout: post
date: 2020-03-01
tag: 
    - Zookeeper
    - Distributed Systems
    - Coordination Service
    - Znodes
    - Quorum
description: "This blog goes little in depth of Zookeeper concepts"
image: https://upload.wikimedia.org/wikipedia/en/thumb/8/81/Apache_ZooKeeper_Logo.svg/1024px-Apache_ZooKeeper_Logo.svg.png
headerImage: https://upload.wikimedia.org/wikipedia/en/thumb/8/81/Apache_ZooKeeper_Logo.svg/1024px-Apache_ZooKeeper_Logo.svg.png
category: blog
author: pranjal
---
### Welcome Back!

This is a follow up blog post to [Zookeeper - All you need to know][1]. If you have not read that blog post, I would highly recommend to go check that out first.

## Modes in Zookeeper

Zookeeper can run in two modes:

- Standalone: Only one server (can be used in development environment)
- Quorum Mode/Ensemble Mode: `2F + 1` servers, where `F` is the number of server failures one can tolerate

## How to configure a Zookeeper Ensemble

- Each Server has a configuration file associated with itself
- Each configuration file lists all the servers that make up the ensemble
- Each server is identified by an id called `sid` or Server Identifier
- `myid` file: tells the server what its id is supposed to be
- `myid` file is kept inside the `dataDir` for each server
- `clientPort` is where clients connect. Mention this in the `connectionString`
- Information related to ensemble is provided in `server.n` entries
- Each `server.n` entry specifies the address and port numbers used by the servers
  - Example - `server.5=127.0.0.1:2222:2223`
  - This example indicates that server with `myid`=5 (S5) will use 2222 for communication and 2223 for leader election. (Shown in the logs below)

## What happens when a Zookeeper server is started?

#### Standalone Mode

Keep calm and let clients connect

#### Ensemble Mode

1. When a server is started, it looks frantically for other servers mentioned in its config file

2. As long as its the only server up and running it will keep throwing `java.net.ConnectException`

3. As soon as another server joins, it proposes an election to take place

4. The following logs will be printed to the console: 

   ```
   My election bind port: /127.0.0.1:2223
   LOOKING
   New election. My id =  5, proposed zxid=0xf00000002
   ```

5. Each server notifies other servers of its proposed leader using a 'Notification Message'

## Notification Message

Notifications are messages that let other peers know that a given peer has changed its vote, either because it has joined leader election or because it learned of another peer with higher `zxid` or same `zxid` and higher server id

Examples -

```
Notification: 1 (message format version), 1 (n.leader), 0x900000003 (n.zxid), 0x1 (n.round), LOOKING (n.state), 1 (n.sid), 0x9 (n.peerEpoch) LEADING (my state)
```

```
Notification: 1 (message format version), 3 (n.leader), 0xa00000001 (n.zxid), 0x1 (n.round), LOOKING (n.state), 1 (n.sid), 0xa (n.peerEpoch) LEADING (my state)
```

```
Notification: 1 (message format version), 5 (n.leader), 0xf00000002 (n.zxid), 0x1 (n.round), LOOKING (n.state), 5 (n.sid), 0xf (n.peerEpoch) LOOKING (my state)
```

```
Notification: 1 (message format version), 2 (n.leader), 0xd00000000 (n.zxid), 0x1 (n.round), FOLLOWING (n.state), 1 (n.sid), 0xf (n.peerEpoch) LOOKING (my state)
```

#### Reading the Notification Message / Notification Log

- `n.sid` is the server which sent the notification
- `n.leader` is the proposed leader; this can be the actual leader in case one exists
- `n.state` is the state of the sender (`sid`)
- `n.zxid` is the timestamp of the last change of the proposed leader

So, this is how you should read a notification message, taking 3rd and 4th example from above:

- Server 5 (S5) is currently in `LOOKING` state and proposes itself as the leader 
- Server 1 (S1) is currently in `FOLLOWING` state and proposes Server 2 (S2) as the leader (maybe S2 is already the `LEADER` since S1 is in `FOLLOWING` state)

## Types of Servers in Zookeeper

- Leader
- Follower
- Observer: Observers do not participate in an election. They are used for High Availability

> Leader, Follower are also called Participants since they participate in an election
>
> Follower, Observer are also called Learners

## More about messages

`LEADER` sends following types of messages:

1. `SNAP` entire snapshot transfer
2. `DIFF` contains most recent missing transactions
3. `INFORM` is information for Observer server
4. `PROPOSAL` proposes state changes to Followers
5. `COMMIT` asks the Followers to commit the changes conveyed in `PROPOSAL`
6. `HEARTBEAT`

#### Let us understand these messages

<img src="../assets/zookeeper-season-2/image01.jpg" alt="Messages exchanged when writing data or state change" style="zoom: 67%;" />
<figcaption class="caption">Messages exchanged when writing data or state change</figcaption>

------

<img src="../assets/zookeeper-season-2/image02.jpg" alt="Messages exchanged after election" style="zoom:67%;" />
<figcaption class="caption">Messages exchanged when syncing data after election</figcaption>

------

<img src="..\assets\zookeeper-season-2\image03.jpg" alt="Leader pings every half tick" style="zoom:67%;" />
<figcaption class="caption">Leader PINGs every half-tick</figcaption>

## Zookeeper in 14 steps

Since you have been so patient in understanding and reading about all these concepts, this is where you are going to reward yourself by making sense of it all.

1. A server connects in `LOOKING` state
2. Election takes place
3. `LEADER` is elected
4. `FOLLOWER` syncs with `LEADER`
   - Based on how much a `FOLLOWER` lags behind, the `LEADER` sends `DIFF` (most recent missing transactions) or a `SNAP` (entire snapshot transfer)
   - Send `INFORM` message to `OBSERVER`
5. Client connects to `FOLLOWER`/`OBSERVER` (also called `LEARNER`)
6. `LERNER` forwards session information of the client to the `LEADER`
7. `LERNER` responds to read requests locally
8. Client sends write request to the `LERNER`
9. `LERNER` forwards write requests to the `LEADER`
10. `LEADER` transforms the request into a transaction
11. `LEADER` sends a `PROPOSE`s the`FOLLOWER` about the transaction
12. `FOLLOWER` responds with ACK
13. If majority of ensemble replies with ACK, `LEADER` sends a `COMMIT` to all the `FOLLOWER`
14. `LEADER` sends `INFORM` message to `OBSERVER`

#### Caveat to point 12

Before acknowledging the `PROPOSAL`, the `FOLLOWER` needs to perform these checks:

- `PROPOSAL` is from the `LEADER` it is currently following
- ACK and `COMMIT` transactions in the same order as broadcasted by the `LEADER`

## Miscellaneous points

- `tick` is the basic unit of measurement for time used by Zookeeper
-  `LEADER` PINGs every half-tick
- Availability depends on Quorum
- Observers do not take part in Quorum
- `epoch` number increases after each election
- Client can not connect to a server that has not seen an update that the client might have seen. i.e. `zxid` of client should be less than or equal to `zxid` of server.

> In a single machine, if a process fails, other processes can detect the failure from OS. However, in a distributed system, the processes which are still running are responsible to detect failure of other processes

## What next?

I have done several experiments with Zookeeper. Experiments with different configurations, added weights to each vote and more scenarios.  Sharing all experiments is not feasible here, but depending on the response to this Zookeeper series of two posts, I may write another follow-up blog in the future regarding one or two of my experiments. Till then, adios amigos! 

[1]: {{site.url}}/Zookeeper-All-you-need-to-know