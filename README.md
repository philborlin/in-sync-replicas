# in-sync-replicas
An in sync replica implementation in go meant to be embedded in distributed systems

# what is it?
In Sync Replicas (ISR) are a way of replicating partitions of data across multiple nodes in an efficient manner.
They are useful for systems that have some natural partioning properties (like a message broker with multiple
topics/streams) where it is desirable to set a replication factor that is less than the total nodes in the cluster.

In a simpler system you can use a consensus algorithm, such as Raft, to replicate a single partition of data.
As you start adding additional partitions the Raft algorithm starts being inefficient because of the number of
messages created when multiple Raft instances are run. In Sync Replicas solve this problem by creating a metadata
controller which uses Raft. This keeps track of all of the partitions along with metadata of each partition such
as who the leader is, which nodes are part of the replica, and which nodes are in sync.

The genius here is the concept of tracking which replicas are in sync. An in-sync replica is a replica that in the
last n milliseconds (a tuneable parameter) was caught up with the leader. When writing to a partition, the write isn't
committed until every node in the in sync replica list is updated. Reads can only come from members of the in sync
replication list. This allows us to stay performant in the face of stuck, lagging, or bootstrapping replicas.

Kafka created this pattern and this implementation is entirely indebted to the hard work of a multitude
of Kafka committers.

# problem
There are several systems written in Go that are re-inventing the wheel
* Jocko - Kafka implemented in Golang with built-in coordination (No ZK dep, single binary install, Cloud Native)
* Liftbridge - Lightweight, fault-tolerant message streams
* committed - A distributed commit log for creating CQRS systems

This embedded server is designed to create a distributed systems primitive that can be shared by systems
interested in using In Sync Replicas. The hope is that this will be as useful as Hashicorps wonderful RAFT
library which is a stellar example what distributed systems primitive should look like.