# Apache Pulsar

## Claims

The main claims that I am interested in are:

- guarantees of no message loss (if recommended configuration applied and your whole data center doesn't burn to the ground)
- strong ordering guarantees
- predictable read and write latency

Apache Pulsar chooses consistency over availability as does its sister projects BookKeeper and ZooKeeper. Every effort is made to give strong consistency.

We'll be taking a look at Pulsar's design to see if those claims are valid. In the next post we'll put the implementation of that design to the test. I won’t cover geo-replication in this post, we’ll look at that another day, we’ll just focus on a single cluster.

## Multiple layers of abstraction

Apache Pulsar has the high level concept of topics and subscriptions and at its lowest level data is stored in binary files which interleave data from multiple topics distributed across multiple servers. In between are a myriad of details and moving parts. I personally find it easier to understand the Pulsar architecture if I separate it out into different layers of abstraction, so that’s what I’ll do in this post.

Let's take a journey down the layers.

View fullsizeFig 1. Layers of abstraction

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538489997612-OYXFLJVUBYP4TOQDPHW9/LayersOfAbstraction.png)

## Layer 1 - Topics, Subscriptions and Cursors

This is not a post about messaging architectures that you can build with Apache Pulsar. We’ll just cover the basics of what topics, subscriptions and cursors are but not any depth about the wider messaging patterns that Pulsar enables.

View fullsizeFig 2. Topics and Subscriptions

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538490975658-WSDKSTF2F07QFPMCPJ5H/TopicsSubscriptions.png)

Messages are stored in topics. A topic, logically, is a log structure with each message being at an offset. Apache Pulsar uses the term Cursor to describe the tracking of offsets. Producers send their messages to a given topic and Pulsar guarantees that once the message has been acknowledged it won’t be lost (bar some super bad catastrophe or poor configuration).

Producers can be of the following types:

- Shared - multiple producers can publish to the same topic
- Exclusive - only one producer can publish to a given topic at a time. A producer will fail if it connects and there is already a producer on the same topic.
- WaitForExclusive - same as exclusive except a producer will simply wait until the topic becomes free and it gets made the exclusive producer.

A consumer consumes messages from a topic via a subscription. A subscription is a logical entity that keeps track of the cursor (the current consumer offset) and also provides some extra guarantees depending on the subscription type:

- Exclusive Subscription - Only one consumer can read the topic via the subscription at a time
- Shared Subscription - Competing consumers can read the topic via the same subscription at the same time.
- Key Shared Subscription - Multiple consumers can read the topic but are not competing consumers, instead the key space is divided between them. This means messages of any given key are always delivered to the same consumer.
- Fail-Over Subscription - Active/Backup pattern for consumers. If the active consumer dies, then the back up takes over. But there are never two active consumers at the same time.

One topic can have multiple attached subscriptions. The subscriptions do not contain the data, only meta-data and a cursor.

Pulsar provides both queueing and log semantics by allowing consumers to treat a Pulsar topic like a queue that deletes messages after being acknowledged by a consumer, or like a log where consumers can rewind their cursor if they want to. Underneath the storage model is the same - a log.

If no data retention policy is set on a topic (via its namespace) then messages are deleted once all cursors of attached subscriptions have passed its offset. That is, the message has been acknowledged on all subscriptions attached to that topic.

However, if a data retention policy exists that covers the topic, then messages are removed once they pass the policy boundary (size of topic, time in topic).

Messages can also be sent with an expiration. These messages are deleted if they exceed the TTL while still unacknowledged. This means that they can be deleted before any consumer gets the chance to read them. Expiration only applies to unacknowledged messages and therefore fits more into the queuing semantics side of things.

TTLs apply to each subscription separately, meaning that “deletion” is a logical deletion. The actual deletion will occur later according to what happens in other subscriptions and any data retention policy.

Consumers acknowledge their messages either one by one, or cumulatively. Cumulative acknowledgement will be better for throughput but introduces duplicate message processing after consumer failures. However, cumulative acknowledgement is not available for shared subscriptions as acknowledgements are based on the offset. However, the consumer API does allow for [batched acknowledgements](http://pulsar.apache.org/api/client/org/apache/pulsar/client/api/ConsumerBuilder.html#acknowledgmentGroupTime-long-java.util.concurrent.TimeUnit-) that will end up with the same number of acks but with fewer RPC calls. This can improve throughput for competing consumers on a shared subscription.

Finally there are partitioned topics similar to the topics of Kafka. The difference is that the partitions in Pulsar are also topics. Just like with Kafka a producer can send messages round-robin, use a hashing algorithm or choose a partition explicitly.

That was a whirlwind introduction to the high-level concepts, we’ll now delve deeper. Remember this is not a primer on Apache Pulsar from 10,000 feet but a look at how it all works underneath from 1000 feet.

## Layer 2 - Logical Storage Model

Now Apache BookKeeper enters the scene. I will talk about BookKeeper in the context of Apache Pulsar, though BookKeeper is a general purpose log storage solution.

First of all, BookKeeper stores data across a cluster of nodes. Each BookKeeper node is called a *bookie*. Secondly, Apache Zookeeper is used by both Pulsar and BookKeeper for storing meta-data and monitoring node health.

View fullsizeFig 3. Apache Pulsar, BookKeeper and ZooKeeper working together

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538572211110-2C6EPHJTLLAOTVMVHXM1/PulsarBkZkCluster.png)

A topic is in fact a stream of *ledgers*. A ledger is a log in its own right. So we compose a parent log (the topic) from a sequence of child logs (ledgers).

Ledgers are appended to a topic, and entries (messages or groups of messages) are appended to ledgers. Ledgers, once closed, are immutable. Ledgers are deleted as a unit, that is, we cannot delete individual entries but ledgers as a whole.

Ledgers themselves are also broken down into *fragments*. Fragments are the smallest unit of distribution across a BookKeeper cluster.

View fullsizeFig 4. Entries at the bottom

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538497275047-DGDCYE3LX945H79I1MOI/LogLedgerFragmentEntry.png)

Topics are a Pulsar concept. *Ledgers*, *fragments* and *entries* are BookKeeper concepts, though Pulsar understands and works with ledgers and entries.

Each ledger (consisting of one or more fragments) can be replicated across multiple BookKeeper nodes (bookies) for both redundancy and read performance. Each fragment is replicated across a different set of bookies (if enough bookies exist).

View fullsizeFig 5. Apache Pulsar, Apache BookKeeper and Apache Zookeeper working together

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538497570494-EIP81VKC3W4VZQBWCWWH/SubTopicLedgerFragmentBookie.png)

Each Ledger has three key configurations:

- Ensemble size (E)
- Write quorum size (Qw)
- Ack quorum size (Qa)

These configurations are applied at the topic level, which Pulsar then sets on the BookKeeper ledgers/fragments of the topic.

Note: "Ensemble" means the actual list of bookies that will be written to. Ensemble size is an instruction to Pulsar to say how big an ensemble it should create. Note that you will need at least E bookies available for writes. By default, bookies are picked up randomly from the list of available bookies (each bookie registers itself in Zookeeper).

There's also the option to configure rack-awareness, by marking bookies as belonging to specific racks. A rack can be a logical construct (eg: an availability zone in a cloud environment). With a rack-aware policy, the BookKeeper client of the Pulsar broker will try to pick bookies from different racks. It's also possible to plug in a custom policy to perform a different type of selection.

Ensemble Size (E) governs the size of the pool of bookies available for that ledger to be written to by Pulsar. Each fragment may have a different ensemble, the broker will select a set of bookies on creating the fragment, but the ensemble will always be the size indicated by E. There must be enough bookies that are write available to cover E.

Write Quorum (Qw) is the number of actual bookies that Pulsar will write an entry to. It can be equal to or smaller than E. The set of bookies a given entry is written to is referred to as its *write-set*.

View fullsizeFig 6. A fragment of 8 entries stored across an ensemble of 3 with each entry written to 3 bookies.

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538556992096-2Y1LIUC0GI3XKGGM064I/FragmentEnsemble.png)

When Qw is smaller than E then we get striping which distributes reads/writes in such a way that each bookie need only serve a subset of read/write requests. Striping can increase total throughput and lower latency.

View fullsizeFig 7. Striping

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538557218910-2K8O0CLWOD9N7ZXX45JS/FragmentWithStriping.png)

So with an ensemble size of 5 and a write quorum of 3 we have 5 different write-sets.

> Striping not necessarily goodWhile it sounds in theory like it could help performance in fact it has a detrimental impact on read performance. BookKeeper goes to great lengths to ensure sequential reads but striping doesn’t play so nicely with that right now. I recommended setting E = Qw.
> 

Ack Quorum (Qa) is a nuanced topic. It is the number of bookies that must acknowledge the write, for the Pulsar broker to send its acknowledgement to its client. But it is also the minimum guaranteed replication factor. It is possible for some entries to only reach Qa and not Qw.

> For an in-depth look at why Qa is the minimum guaranteed replication factor, I have written about it in detail on the Splunk MaaS blog.
> 

In practice you would set Qa as either:

- (Qa == Qw) or
- (Qa == Qw -1) ---> This will reduce publish latency and reduce memory usage in Pulsar.

A ledger is created on creating a new topic or when roll-over occurs. Roll-over is the concept of creating a new ledger when either:

- a ledger size or time limit has been reached
- ownership (by a Pulsar broker) of a topic changes (more on that later).

A fragment is created when:

- a new ledger is created
- when a write to a bookie fails.

When a bookie cannot serve a write then the Pulsar broker gets busy creating a new fragment and making sure the write gets acknowledged by Qw bookies. It’s like the Terminator, it won’t stop until that message is persisted.

> Scaling benefits of segmented logsAdding new bookies does not mean manual rebalancing needs to be performed. Automatically, those new bookies will be candidates for new fragments. After joining the cluster, new bookies will be written to immediately upon new fragments/ledgers being created. Each fragment can be stored on a different subset of bookies in the cluster! We do not couple topics or ledgers to a given bookie or set of bookies.
> 

Let’s stop and take stock. This is a very different and more complex model to Kafka. With Kafka each partition replica is stored in its entirety on a single broker (tiered storage is coming to OSS though). The partition replica is comprised of a series of segment and index files. This [blog post](https://thehoard.blog/how-kafkas-storage-internals-work-3a29b02e026) nicely describes it.

The great thing about the Kafka model is that it is simple and fast. All reads and writes are sequential. The bad thing is that a single broker must have enough storage to cope with that replica, so very large replicas can force you to have very large disks. The second downside is that rebalancing partitions when you grow your cluster becomes necessary. This can be painful and requires good planning and execution to pull if off without any hitches.

Returning to the Pulsar + BookKeeper model. The data of a given topic is spread across multiple bookies. The topic has been split into ledgers and the ledgers into fragments. When you need to grow your cluster, just add more bookies and they’ll start getting written to when new fragments are created. No more Kafka-style rebalancing required. Reads and writes now have to jump around a bit between bookies, which isn’t a bad thing. We’ll see how Pulsar manages this and does it fast further down this post.

Each Pulsar broker needs to keep track of the ledgers and fragments that each topic is comprised of. This metadata is stored in ZooKeeper.

In the storage layer we've written a topic evenly across a BookKeeper cluster. We've avoided the pitfalls of coupling topic replicas to specific nodes. Where Kafka topics are like sticks of Toblerone, our Pulsar topics are like a gas expanding to fill the available space. This avoids painful rebalancing.

## Layer 2 - Pulsar Brokers and Topic Ownership

Also in Layer 2 of my abstraction layers we have the Pulsar brokers. Pulsar brokers have no persistent state that cannot be lost. They are separated from the storage layer. A BookKeeper cluster by itself does not perform replication, each bookie is just a follower that is told what to do by a leader - the leader being a Pulsar broker. Each topic is owned by a single Pulsar broker. That broker serves all reads and writes of that topic.

When a Pulsar broker receives a write, it will perform that write against the ensemble of the current fragment of that topic. Remember that if no striping occurs the ensemble of each entry is the same as the fragment ensemble. If striping occurs then each entry has its own ensemble which is a subset of the fragment ensemble.

In a normal situation there will be a single fragment in the current ledger. Once Qa brokers have acknowledged the write the Pulsar broker will send an acknowledgement to the producer client.

An acknowledgement can only be sent if all prior messages have also been Qa acknowledged. If for a given message, a Bookie responds with an error or does not respond at all, then the broker will create a new fragment on a new ensemble of bookies (that does not include the problem bookie).

View fullsizeFig 8. A single broker serves all reads and writes of a given topic.

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538559874547-UYUESUUV7Q33RK5BJXDB/PulsarBrokerWrites.png)

Note that the broker will only wait for Qa acks from the bookies.

Reads also go through the owner. The broker, being the singular entrypoint for a given topic, knows up to which offset has been safely persisted to BookKeeper. It needs only read from a single bookie to serve a read. We’ll see in Layer 3 how it uses caching to serve many reads from its in-memory cache rather than sending reads to BookKeeper.

View fullsizeFig 9. Reads only need go to a single Bookie

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538560169571-MKFXQQ8FJ6MMJH4V59EO/PulsarBrokerReads.png)

Pulsar Broker health is monitored by ZooKeeper. When a broker fails or becomes unavailable (to ZooKeeper) an ownership change occurs. A new broker becomes the topic owner and all clients are now directed to read/write to this new broker.

BookKeeper has a critically important functionality called Fencing. Fencing allows BookKeeper to guarantee that only one writer (Pulsar broker) can be writing to a ledger.

> Last Added Confirmed (LAC)The LAC is the commit index of a ledger. No reads should pass this point as they will be dirty reads. There are no consistency guarantees beyond the LAC and for correctness, no reads should read past it.
> 

It works as follows:

1. The current Pulsar broker (B1) that has ownership of topic X is deemed dead or unavailable (via ZooKeeper).
2. Another broker (B2) updates the state of the current ledger of topic X to IN_RECOVERY from OPEN.
3. B2 sends a fencing LAC read request to all bookies of the current fragment of the ledger and waits for (Qw-Qa)+1 responses. Once this number of responses is received the ledger is now fenced. The old broker if it is in fact still alive, can no longer make writes as it will not be able to get Qa acknowledgements (due to fencing exception responses).
4. B2 takes the highest LAC response and then starts performing recovery reads from the LAC + 1. It ensures that all entries from that point on (which may not have been previously acknowledged to the Pulsar broker) get replicated to Qw bookies. Once B2 cannot read and replicate any more entries, the ledger is fully recovered.
5. B2 changes the state of the ledger to CLOSED
6. B2 opens a new ledger and can now accept writes to the topic.

The great thing about this architecture is that by making the leaders (the Pulsar brokers) have no state, split-brain is trivially taken care of by BookKeeper's fencing functionality. There is no split-brain, no divergence, no data loss.

## Layer 2 - Cursor Tracking

Each subscription stores a cursor. The cursor is the current offset in the log. Subscriptions store their cursor in BookKeeper in ledgers. This makes cursor tracking scalable just like topics.

## Layer 3 - Bookie Storage

Ledgers and fragments are logical constructs which are maintained and tracked in ZooKeeper. Physically, the data is not stored in files that correspond to ledgers and fragments. The actual implementation of storage in BookKeeper is pluggable and Pulsar uses a storage implementation called DbLedgerStorage by default.

When a write to a bookie occurs, first that message is written to a journal file. This is a write-ahead log (WAL) and it helps BookKeeper avoid data loss in the event of a failure. It is the same mechanism by which relational databases achieve their durability guarantees.

The write is also made to the *write cache*. The write cache accumulates writes and periodically sorts and flushes them to disk in *entry log files*. Writes are sorted so that entries of the same ledger are placed together which improves read performance. If the entries are written in strict temporal order then reads will not benefit from a sequential layout on disk. By aggregating and sorting we achieve temporal ordering at the ledger level which is what we care about.

The write cache also writes the entries to RocksDB which stores an index of the location of each entry. It simply maps (ledgerId, entryId) to (entryLogId, offset in the file).

Reads hit the write cache first as the write cache has the latest messages. If there is a cache miss then it hits the read cache. If there is a second cache-miss then the location of the requested entry is looked up in RocksDB and then reads that entry in the correct entry log file. It performs a read-ahead and updates the read cache so that following requests are more likely to get a cache hit. These two layers of caching mean that the vast majority reads are generally served from memory.

BookKeeper allows you to isolate disk IO from reads and writes. Writes are all written sequentially to the journal file that can be stored on a dedicated disk and are committed in groups for even greater throughput. After that no other disk IO is synchronous from the point of view of the writer. Data is just written to memory buffers.

Asynchronously on background threads, the write cache performs bulk writes to entry log files and RocksDB, which typically run a their own shared disk. So one disk for synchronous writes (journal file) and another disk for asynchronous optimized writes and all reads.

On the read-side, readers are served from either the read cache or from the log entry files and RocksDB.

Also take into account that writes can saturate the ingress network bandwidth and reads can saturate the egress network bandwidth, but they do not affect each other.

This elegantly isolated reads from writes at a disk and network level.

View fullsizeFig 10. A Bookie with the default (with Apache Pulsar) DbLedgerStorage architecture.

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538601042753-J8IXQ06HUPV1R7PEAPFE/image-asset.png)

## Layer 3 - Pulsar Broker Caching

Each topic has a single broker that acts as owner. All reads and writes go through that broker. This provides many benefits.

Firstly, the broker can cache the log head in memory meaning that the broker can serve tailing readers itself without the need for BookKeeper. This avoids paying the cost of a network round-trip and a possible disk read on a bookie.

The broker is also aware of the id of the Last Add Confirmed (LAC) entry. It can track which message is the last safely persisted message.

When the broker does not have the message in its cache it will request the data from one bookie in the ensemble of the fragment of that message. This means that the difference in read serving performance between tail readers and catch-up readers is large. Tail readers can be served from memory on the Pulsar broker whereas a catch-up reader may have to incur the cost of an extra network round trip and multiple disk reads if neither the write nor read Cache have the data.

So we’ve covered from a high level the logical and physical representation of messages, as well as the different actors in a Pulsar cluster and their relationships with each other.  There is plenty of detail that has not been covered but we’ll leave that as an exercise for another day.

Next up we’ll cover how an Apache Pulsar cluster ensures that messages are sufficiently replicated after node failures.

## Manual/Auto Recovery

When a bookie fails, all the ledgers that have fragments on that bookie are now under replicated. Recovery is the process of "rereplicating" fragments to ensure the replication factor (Qw) is maintained for each ledger.

> This recovery mechanism should not be confused with Ledger Recovery which is part of the replication protocol with fencing where a broker can safely close the ledger of another broker.Manual/auto recovery is not part of the BookKeeper replication protocol but external to it and used as an asynchronous repair mechanism.
> 

There are two types of recovery: manual or automatic. The rereplication protocol is the same for both, but Automatic Recovery uses an in-built failed node detection mechanism that registers rereplication tasks to be performed. The manual process requires manual intervention.

We'll focus on the Auto Recovery mode.

Auto Recovery can be run from a dedicated set of servers or hosted on the bookies, in the AutoRecoveryMain process. One of the auto-recovery processes gets elected as Auditor. The role of the Auditor is to detect downed bookies and then:

1. Read the full ledger list from ZK and find the ledgers hosted on the failed bookie.
2. For each ledger it will create a rereplication task in the /underreplicated znode in ZooKeeper.

If the Auditor node fails then another node gets promoted as the Auditor. The Auditor is a thread in the AutoRecoveryMain process.

The AutoRecoveryMain process also has a thread that runs a Replication Task Worker. Each worker watches the /underreplicated znode for tasks.

On seeing a task it will try and lock it. If it is not able to acquire the lock, it will move onto the next task.

If it does manage to acquire a lock it then:

1. Scans the ledger for fragments which its local bookie is not a member of
2. For each matching fragment, it replicates the data from another bookie to its own bookie, updates ZooKeeper with the new ensemble and the fragment is marked as fully replicated.

If the ledger has remaining underreplicated fragments then the lock is released. If all fragments are all fully replicated the task is deleted from /underreplicated.

If a fragment does not have an end entry id then the replication task waits and checks again, if the fragment still has no end entry id it fences the ledger before rereplicating the fragment.

Therefore, with Auto Recovery mode, a Pulsar cluster is able to self-repair in the face of storage layer failures. The admin must just ensure that the right amount of bookies are deployed.

## ZooKeeper

ZooKeeper is required by both Pulsar and BookKeeper. If a Pulsar node loses visibility of all ZooKeeper nodes then it stops accepting read and writes and restarts itself. This is as a precaution to ensure that the cluster cannot enter an inconsistent state.

This does mean that if ZooKeeper goes down, everything becomes unavailable and that all Pulsar node caches will be wiped. Therefore upon resumption of service there could in theory be a latency spike due to all reads going to BookKeeper.

## Round Up

- Each topic has an owner broker
- Each topic is logically broken down into ledgers, fragments and entries
- Fragments are distributed across the bookie cluster. There is no coupling of a given topic to a given bookie(s).
- Fragments can be striped across multiple bookies.
- When a Pulsar broker fails, ownership of the topics of that broker fail-over to another broker. Fencing avoids two brokers that might believe themselves the owner from actually writing to the current topic ledger at the same time.
- When a bookie fails, auto recovery (if enabled) will automatically perform “rereplication” of the data to other bookies. If disabled, a manual process can be initiated
- Brokers cache the log head allowing them to serve tailing readers very efficiently
- Bookies use a journal to provide guarantees on failure. The journal can be used to recover data not yet written to entry log files at the time of the failure.
- Entries of all topics are interleaved in entry log files. A lookup index is kept in RocksDB.
- Bookies serve reads as follows: write cache -> read cache -> log entry files
- Bookies can isolate reads from writes IO via separate disks for journal files, log entry files and RocksDB.
- ZooKeeper stores all meta-data for both Pulsar and BookKeeper. If ZooKeeper is unavailable Pulsar is unavailable.
- Storage can be scaled out separately to the Pulsar brokers. If storage is the bottleneck then simply add more bookies and they will start taking on load without the need for rebalancing.

View fullsizeFig 11. Round-up of concepts

![](https://images.squarespace-cdn.com/content/v1/56894e581c1210fead06f878/1538583413563-QVJ2MWQQK1B10NIP45NZ/RoundUp.png)

## Some notes on data loss

As mentioned earlier, ack quorum is the minimum guaranteed replication factor. So to lose data you must lose Qa bookies. This is why using an ack quorum of 1 is unsafe.

> When you use a majority quorum based system (such as Apache Kafka with acks=all and min-insync-replicas=2 and rep-factor=3) then your minimum guaranteed replication factor is the majority. So with a replication factor of 3, the minimum guarantee is 2. So this is not too dissimilar to BookKeeper. The difference with BookKeeper is you get to control the minimum quorum explicitly.
> 

Because BookKeeper writes to a write-ahead log and fsyncs to disk before acknowledging writes, Pulsar cannot lose data in situations where the entire cluster abruptly fails - such as a power loss event. This gives Pulsar an extra safety edge over Kafka.

## Some notes on availability

Pulsar gets better write availability than other systems like RabbitMQ or Kafka because of the dynamic nature of each topic. A topic is a segmented log, composed of ledgers, and is free is move away from failed bookies. An entire bookie ensemble could go down and Pulsar could carry on by simply closing the current ledger and opening a new one on a functioning set of bookies. Reads that need to be served by BookKeeper would be impacted but as soon as just a single bookie from a fragment’s ensemble is available, reads can continue also.

Using an ack quorum of 1 is unsafe not just because it can leave some entries without redundancy, it also can block ledger recovery. Ledger recovery needs to keep reading until it finds the last committed entry. But with a Qa of 1, if just a single bookie is down, it cannot know for sure that that bookie hosts an entry or not and so recovery will stall.

## Conclusion

There are more details I haven’t included but having been a contributor to Pulsar and BookKeeper for over a year now I have written a number of blog posts that go into the low level details. Those links are at the bottom of this post.

Apache Pulsar is significantly more complicated than Apache Kafka in terms of its protocols and storage model.

The two stand-out features of a Pulsar cluster are:

- Separation of brokers from storage, combined with BookKeepers fencing functionality, elegantly avoids split-brain scenarios that could provoke data loss.
- Breaking topics into ledgers and fragments, and distributing those across a cluster allow Pulsar clusters to scale out with ease. New data automatically starts getting written to new bookies. No rebalancing is required.

Plus I haven’t even gotten to geo-replication and tiered storage which are also amazing features.

My feeling is that Pulsar and BookKeeper are part of the next generation of data streaming systems. Their protocols are well thought out and rather elegant. But with added complexity comes added risk of bugs. In the next post we’ll start chaos testing an Apache Pulsar cluster and see if we can identify weaknesses in the protocols, and any implementation bugs or anomalies.

Posts I’ve written about Pulsar:

- https://jack-vanlightly.com/blog/2018/10/21/how-to-not-lose-messages-on-an-apache-pulsar-cluster
- https://jack-vanlightly.com/blog/2018/10/25/testing-producer-deduplication-in-apache-kafka-and-apache-pulsar
- https://jack-vanlightly.com/blog/2019/9/4/a-look-at-multi-topic-subscriptions-with-apache-pulsar

Posts I’ve written about Apache BookKeeper:

- https://medium.com/splunk-maas/detecting-bugs-in-data-infrastructure-using-formal-methods-704fde527c58
- https://medium.com/splunk-maas/a-guide-to-the-bookkeeper-replication-protocol-tla-series-part-2-29f3371fe395
- https://medium.com/splunk-maas/modelling-and-verifying-the-bookkeeper-protocol-tla-series-part-3-ef8a9850ad63
- https://medium.com/splunk-maas/apache-bookkeeper-internals-part-1-high-level-6dce62269125
- https://medium.com/splunk-maas/apache-bookkeeper-internals-part-2-writes-359ffc17c497
- https://medium.com/splunk-maas/apache-bookkeeper-internals-part-3-reads-31637b118bf
- https://medium.com/splunk-maas/apache-bookkeeper-internals-part-4-back-pressure-7847bd6d1257
- https://medium.com/splunk-maas/apache-bookkeeper-observability-part-1-introducing-the-metrics-7f0acb32d0dc
- https://medium.com/splunk-maas/apache-bookkeeper-observability-part-2-write-use-metrics-f359f2b83539
- https://medium.com/splunk-maas/apache-bookkeeper-observability-part-3-write-metrics-in-detail-178c216b6373
- https://medium.com/splunk-maas/apache-bookkeeper-observability-part-4-read-use-metrics-10faafae0de5
- https://medium.com/splunk-maas/apache-bookkeeper-observability-part-5-read-metrics-in-detail-2f53acac3f7e
- https://medium.com/splunk-maas/apache-bookkeeper-insights-part-1-external-consensus-and-dynamic-membership-c259f388da21
- https://medium.com/splunk-maas/apache-bookkeeper-insights-part-2-closing-ledgers-safely-386a399d0524

Banner image credit: ESO/[J. Girard](http://www.juliengirard.space/) ([djulik.com](http://djulik.com/)). [Link to image](https://www.eso.org/public/images/potw1226a/).