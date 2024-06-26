[TOC]

Overview
------------------------------------------------------------------------------------

Replication is a feature of Accumulo which provides a mechanism to automatically copy data to other systems, typically for the purpose of disaster recovery, high availability, or geographic locality. It is best to consider this feature as a framework for automatic replication instead of the ability to copy data from to another Accumulo instance as copying to another Accumulo cluster is only an implementation detail. The local Accumulo cluster is hereby referred to as the `primary` while systems being replicated to are known as `peers`.

This replication framework makes two Accumulo instances, where one instance replicates to another, eventually consistent between one another, as opposed to the strong consistency that each single Accumulo instance still holds. That is to say, attempts to read data from a table on a peer which has pending replication from the primary will not wait for that data to be replicated before running the scan. This is desirable for a number of reasons, the most important is that the replication framework is not limited by network outages or offline peers, but only by the HDFS space available on the primary system.

Replication configurations can be considered as a directed graph which allows cycles. The systems in which data was replicated from is maintained in each Mutation which allow each system to determine if a peer already has the data in which the system wants to send.

Data is replicated by using the Write-Ahead logs (WAL) that each TabletServer is already maintaining. TabletServers record which WALs have data that need to be replicated to the `accumulo.metadata` table. The Manager uses these records, combined with the local Accumulo table that the WAL was used with, to create records in the `replication` table which track which peers the given WAL should be replicated to. The Manager latter uses these work entries to assign the actual replication task to a local TabletServer using ZooKeeper. A TabletServer will get a lock in ZooKeeper for the replication of this file to a peer, and proceed to replicate to the peer, recording progress in the `replication` table as data is successfully replicated on the peer. Later, the Manager and Garbage Collector will remove records from the `accumulo.metadata` and `replication` tables and files from HDFS, respectively, after replication to all peers is complete.

Configuration
----------------------------------------------------------------------------------------------

Configuration of Accumulo to replicate data to another system can be categorized into the following sections.

### Site Configuration

Each system involved in replication (even the primary) needs a name that uniquely identifies it across all peers in the replication graph. This should be considered fixed for an instance, and set using [replication.name]($Server-Properties-2.x#replication_name) in `accumulo.properties`.

```
# Unique name for this system used by replication
replication.name=primary
```

### Instance Configuration

For each peer of this system, Accumulo needs to know the name of that peer, the class used to replicate data to that system and some configuration information to connect to this remote peer. In the case of Accumulo, this additional data is the Accumulo instance name and ZooKeeper quorum; however, this varies on the replication implementation for the peer.

These can be set in `accumulo.properties` to ease deployments; however, as they may change, it can be useful to set this information using the Accumulo shell.

To configure a peer with the name `peer1` which is an Accumulo system with an instance name of `accumulo_peer` and a ZooKeeper quorum of `10.0.0.1,10.0.2.1,10.0.3.1`, invoke the following command in the shell.

```
root@accumulo_primary> config -s
replication.peer.peer1=org.apache.accumulo.tserver.replication.AccumuloReplicaSystem,accumulo_peer,10.0.0.1,10.0.2.1,10.0.3.1
```

Since this is an Accumulo system, we also want to set a username and password to use when authenticating with this peer. On our peer, we make a special user which has permission to write to the tables we want to replicate data into, “replication” with a password of “password”. We then need to record this in the primary’s configuration.

```
root@accumulo_primary> config -s replication.peer.user.peer1=replication
root@accumulo_primary> config -s replication.peer.password.peer1=password
```

Alternatively, when configuring replication on Accumulo running Kerberos, a keytab file per peer can be configured instead of a password. The provided keytabs must be readable by the unix user running Accumulo. They keytab for a peer can be unique from the keytab used by Accumulo or any keytabs for other peers.

```
accumulo@EXAMPLE.COM@accumulo_primary> config -s replication.peer.user.peer1=replication@EXAMPLE.COM
accumulo@EXAMPLE.COM@accumulo_primary> config -s replication.peer.keytab.peer1=/path/to/replication.keytab
```

### Table Configuration

Now, we presently have a peer defined, so we just need to configure which tables will replicate to that peer. We also need to configure an identifier to determine where this data will be replicated on the peer. Since we’re replicating to another Accumulo cluster, this is a table ID. In this example, we want to enable replication on `my_table` and configure our peer `accumulo_peer` as a target, sending the data to the table with an ID of `2` in `accumulo_peer`.

```
root@accumulo_primary> config -t my_table -s table.replication=true
root@accumulo_primary> config -t my_table -s table.replication.target.accumulo_peer=2
```

To replicate a single table on the primary to multiple peers, the second command in the above shell snippet can be issued, for each peer and remote identifier pair.

Monitoring
----------------------------------------------------------------------------------------

Basic information about replication status from a primary can be found on the Accumulo Monitor server, using the `Replication` link the sidebar.

On this page, information is broken down into the following sections:

1.  Files pending replication by peer and target
2.  Files queued for replication, with progress made

Work Assignment
--------------------------------------------------------------------------------------------------

Depending on the schema of a table, different implementations of the [WorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/WorkAssigner.html) used could be configured. The implementation is controlled via the property [replication.work.assigner]($Server-Properties-2.x#replication_work_assigner) and the full class name for the implementation. This can be configured via the shell or `accumulo.properties`.

Two implementations of [WorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/WorkAssigner.html) are provided:

1.  The [UnorderedWorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-manager/2.1.2/org/apache/accumulo/manager/replication/UnorderedWorkAssigner.html) can be used to overcome the limitation of only a single WAL being replicated to a target and peer at any time. Depending on the table schema, it’s possible that multiple versions of the same Key with different values are infrequent or nonexistent. In this case, parallel replication to a peer and target is possible without any downsides. In the case where this implementation is used were column updates are frequent, it is possible that there will be an inconsistency between the primary and the peer.

2.  The [SequentialWorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-manager/2.1.2/org/apache/accumulo/manager/replication/SequentialWorkAssigner.html) is configured for an instance. The SequentialWorkAssigner ensures that, per peer and each remote identifier, each WAL is replicated in the order in which they were created. This is sufficient to ensure that updates to a table will be replayed in the correct order on the peer. This implementation has the downside of only replicating a single WAL at a time.


ReplicaSystems[](https://accumulo.apache.org/docs/2.x/administration/replication#replicasystems)
------------------------------------------------------------------------------------------------

[ReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/ReplicaSystem.html) is the interface which allows abstraction of replication of data to peers of various types. Presently, only an [AccumuloReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/AccumuloReplicaSystem.html) is provided which will replicate data to another Accumulo instance. A [ReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/ReplicaSystem.html) implementation is run inside of the TabletServer process, and can be configured as mentioned in [Instance Configuration](https://accumulo.apache.org/docs/2.x/administration/replication#instance-configuration) section of this document. Theoretically, an implementation of this interface could send data to other filesystems, databases, etc.

### AccumuloReplicaSystem

The [AccumuloReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/AccumuloReplicaSystem.html) uses Thrift to communicate with a peer Accumulo instance and replicate the necessary data. The TabletServer running on the primary will communicate with the Manager on the peer to request the address of a TabletServer on the peer which this TabletServer will use to replicate the data.

The TabletServer on the primary will then replicate data in batches of a configurable size ([replication.max.unit.size]($Server-Properties-2.x#replication_max_unit_size)). The TabletServer on the peer will report how many records were applied back to the primary, which will be used to record how many records were successfully replicated. The TabletServer on the primary will continue to replicate data in these batches until no more data can be read from the file.

Other Configuration
----------------------------------------------------------------------------------------------------------

There are a number of configuration values that can be used to control how the implementation of various components operate.

*   [replication.max.work.queue]($Server-Properties-2.x#replication_max_work_queue) - Maximum number of files queued for replication at one time
*   [replication.work.assignment.sleep]($Server-Properties-2.x#replication_work_assignment_sleep) - Time between invocations of the [WorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/WorkAssigner.html)
*   [replication.worker.threads]($Server-Properties-2.x#replication_worker_threads) - Size of threadpool used to replicate data to peers
*   [replication.receipt.service.port]($Server-Properties-2.x#replication_receipt_service_port) - Thrift service port to listen for replication requests, can use ‘0’ for a random port
*   [replication.work.attempts]($Server-Properties-2.x#replication_work_attempts) - Number of attempts to replicate to a peer before aborting the attempt
*   [replication.receiver.min.threads]($Server-Properties-2.x#replication_receiver_min_threads) - Minimum number of idle threads for handling incoming replication
*   [replication.receiver.threadcheck.time]($Server-Properties-2.x#replication_receiver_threadcheck_time) - Time between attempting adjustments of thread pool for incoming replications
*   [replication.max.unit.size]($Server-Properties-2.x#replication_max_unit_size) - Maximum amount of data to be replicated in one RPC
*   [replication.work.assigner]($Server-Properties-2.x#replication_work_assigner) - [WorkAssigner](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/WorkAssigner.html) implementation
*   [tserver.replication.batchwriter.replayer.memory]($Server-Properties-2.x#tserver_replication_batchwriter_replayer_memory) - Size of BatchWriter cache to use in applying replication requests

Example Practical Configuration
----------------------------------------------------------------------------------------------------------------------------------

A real-life example is now provided to give concrete application of replication configuration. This example is a two instance Accumulo system, one primary system and one peer system. They are called **primary** and **peer**, respectively. Each system also have a table of the same name, `my_table`. The instance name for each is also the same (`primary` and `peer`), and both have ZooKeeper hosts on a node with a hostname with that name as well (primary:2181 and peer:2181).

We want to configure these systems so that `my_table` on **primary** replicates to `my_table` on **peer**.

### accumulo.properties

We can assign the “unique” name that identifies this Accumulo instance among all others that might participate in replication together. In this example, we will use the names provided in the description.

#### Primary

```xml
replication.name=primary
```

#### Peer

```xml
replication.name=peer
```

### managers and tservers files

Be _sure_ to use non-local IP addresses. Other nodes need to connect to it and using localhost will likely result in a local node talking to another local node.

### Start both instances

The rest of the configuration is dynamic and is best configured on the fly (in ZooKeeper) than in accumulo.properties.

### Peer

The next series of command are to be run on the peer system. Create a user account for the primary instance called “peer”. The password for this account will need to be saved in the configuration on the primary

```
root@peer> createtable my_table
root@peer> createuser peer
root@peer> grant -t my_table -u peer Table.WRITE
root@peer> grant -t my_table -u peer Table.READ
root@peer> tables -l
```

Remember what the table ID for ‘my\_table’ is. You’ll need that to configure the primary instance.

### Primary

Next, configure the primary instance.

#### Set up the table

```
root@primary> createtable my_table
```

#### Define the Peer as a replication peer to the Primary

We’re defining the instance with [replication.name]($Server-Properties-2.x#replication_name) of `peer` as a peer. We provide the implementation of [ReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/ReplicaSystem.html) that we want to use, and the configuration for the [AccumuloReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/AccumuloReplicaSystem.html). In this case, the configuration is the Accumulo Instance name for `peer` and the ZooKeeper quorum string. The configuration key is of the form `replication.peer.$peer_name`.

```
root@primary> config -s replication.peer.peer=org.apache.accumulo.tserver.replication.AccumuloReplicaSystem,peer,$peer_zk_quorum
```

#### Set the authentication credentials

We want to use that special username and password that we created on the peer, so we have a means to write data to the table that we want to replicate to. The configuration key is of the form “replication.peer.user.$peer\_name”.

```
root@primary> config -s replication.peer.user.peer=peer
root@primary> config -s replication.peer.password.peer=peer
```

#### Enable replication on the table

Now that we have defined the peer on the primary and provided the authentication credentials, we need to configure our table with the implementation of [ReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/ReplicaSystem.html) we want to use to replicate to the peer. In this case, our peer is an Accumulo instance, so we want to use the [AccumuloReplicaSystem](https://static.javadoc.io/org.apache.accumulo/accumulo-server-base/2.1.2/org/apache/accumulo/server/replication/AccumuloReplicaSystem.html).

The configuration for the AccumuloReplicaSystem is the table ID for the table on the peer instance that we want to replicate into. Be sure to use the correct value for $peer\_table\_id. The configuration key is of the form “table.replication.target.$peer\_name”.

```
root@primary> config -t my_table -s table.replication.target.peer=$peer_table_id
```

Finally, we can enable replication on this table.

```
root@primary> config -t my_table -s table.replication=true
```

While this feature is intended for general-purpose use, its implementation does carry some baggage. Like any software, replication is a feature that operates well within some set of use cases but is not meant to support all use cases. For the benefit of the users, we can enumerate these cases.

### Latency

As previously mentioned, the replication feature uses the Write-Ahead Log files for a number of reasons, one of which is to prevent the need for data to be written to RFiles before it is available to be replicated. While this can help reduce the latency for a batch of Mutations that have been written to Accumulo, the latency is at least seconds to tens of seconds for replication once ingest is active. For a table which replication has just been enabled on, this is likely to take a few minutes before replication will begin.

Once ingest is active and flowing into the system at a regular rate, replication should be occurring at a similar rate, given sufficient computing resources. Replication attempts to copy data at a rate that is to be considered low latency but is not a replacement for custom indexing code which can ensure near real-time referential integrity on secondary indexes.

### Table-Configured Iterators

Accumulo Iterators tend to be a heavy hammer which can be used to solve a variety of problems. In general, it is highly recommended that Iterators which are applied at major compaction time are both idempotent and associative due to the non-determinism in which some set of files for a Tablet might be compacted. In practice, this translates to common patterns, such as aggregation, which are implemented in a manner resilient to duplication (such as using a Set instead of a List).

Due to the asynchronous nature of replication and the expectation that hardware failures and network partitions will exist, it is generally not recommended to not configure replication on a table which has Iterators set which are not idempotent. While the replication implementation can make some simple assertions to try to avoid re-replication of data, it is not presently guaranteed that all data will only be sent to a peer once. Data will be replicated at least once. Typically, this is not a problem as the VersioningIterator will automatically deduplicate this over-replication because they will have the same timestamp; however, certain Combiners may result in inaccurate aggregations.

As a concrete example, consider a table which has the SummingCombiner configured to sum all values for multiple versions of the same Key. For some key, consider a set of numeric values that are written to a table on the primary: \[1, 2, 3\]. On the primary, all of these are successfully written and thus the current value for the given key would be 6, (1 + 2 + 3). Consider, however, that each of these updates to the peer were done independently (because other data was also included in the write-ahead log that needed to be replicated). The update with a value of 1 was successfully replicated, and then we attempted to replicate the update with a value of 2 but the remote server never responded. The primary does not know whether the update with a value of 2 was actually applied or not, so the only recourse is to re-send the update. After we receive confirmation that the update with a value of 2 was replicated, we will then replicate the update with 3. If the peer did never apply the first update of ‘2’, the summation is accurate. If the update was applied but the acknowledgement was lost for some reason (system failure, network partition), the update will be resent to the peer. Because addition is non-idempotent, we have created an inconsistency between the primary and peer. As such, the SummingCombiner wouldn’t be recommended on a table being replicated.

While there are changes that could be made to the replication implementation which could attempt to mitigate this risk, presently, it is not recommended to configure Iterators or Combiners which are not idempotent to support cases where inaccuracy of aggregations is not acceptable.

### Duplicate Keys

In Accumulo, when more than one key exists that are exactly the same, keys that are equal down to the timestamp, the retained value is non-deterministic. Replication introduces another level of non-determinism in this case. For a table that is being replicated and has multiple equal keys with different values inserted into it, the final value in that table on the primary instance is not guaranteed to be the final value on all replicas.

For example, say the values that were inserted on the primary instance were `value1` and `value2` and the final value was `value1`, it is not guaranteed that all replicas will have `value1` like the primary. The final value is non-deterministic for each instance.

As is the recommendation without replication enabled, if multiple values for the same key (sans timestamp) are written to Accumulo, it is strongly recommended that the value in the timestamp properly reflects the intended version by the client. That is to say, newer values inserted into the table should have larger timestamps. If the time between writing updates to the same key is significant (order minutes), this concern can likely be ignored.

### Bulk Imports

Currently, files that are bulk imported into a table configured for replication are not replicated. There is no technical reason why it was not implemented, it was simply omitted from the initial implementation. This is considered a fair limitation because bulk importing generated files multiple locations is much simpler than bifurcating “live” ingest data into two instances. Given some existing bulk import process which creates files and them imports them into an Accumulo instance, it is trivial to copy those files to a new HDFS instance and import them into another Accumulo instance using the same process. Hadoop’s `distcp` command provides an easy way to copy large amounts of data to another HDFS instance which makes the problem of duplicating bulk imports very easy to solve.

Table Schema
--------------------------------------------------------------------------------------------

The following describes the kinds of keys, their format, and their general function for the purposes of individuals understanding what the replication table describes. Because the replication table is essentially a state machine, this data is often the source of truth for why Accumulo is doing what it is with respect to replication. There are three “sections” in this table: “repl”, “work”, and “order”.

### Repl section

This section is for the tracking of a WAL file that needs to be replicated to one or more Accumulo remote tables. This entry is tracking that replication needs to happen on the given WAL file, but also that the local Accumulo table, as specified by the column qualifier “local table ID”, has information in this WAL file.

The structure of the key-value is as follows:

```
<HDFS_uri_to_WAL> repl:<local_table_id> [] -> <protobuf>
```

This entry is created based on a replication entry from the Accumulo metadata table, and is deleted from the replication table when the WAL has been fully replicated to all remote Accumulo tables.

### Work section

This section is for the tracking of a WAL file that needs to be replicated to a single Accumulo table in a remote Accumulo cluster. If a WAL must be replicated to multiple tables, there will be multiple entries. The Value for this Key is a serialized ProtocolBuffer message which encapsulates the portion of the WAL which was already sent for this file. The “replication target” is the unique location of where the file needs to be replicated: the identifier for the remote Accumulo cluster and the table ID in that remote Accumulo cluster. The protocol buffer in the value tracks the progress of replication to the remote cluster.

```
<HDFS_uri_to_WAL> work:<replication_target> [] -> <protobuf>
```

The “work” entry is created when a WAL has an “order” entry, and deleted after the WAL is replicated to all necessary remote clusters.

### Order section

This section is used to order and schedule (create) replication work. In some cases, data with the same timestamp may be provided multiple times. In this case, it is important that WALs are replicated in the same order they were created/used. In this case (and in cases where this is not important), the order entry ensures that oldest WALs are processed most quickly and pushed through the replication framework.

```
<time_of_WAL_closing>\x00<HDFS_uri_to_WAL> order:<local_table_id> [] -> <protobuf>
```

The “order” entry is created when the WAL is closed (no longer being written to) and is removed when the WAL is fully replicated to all remote locations.
