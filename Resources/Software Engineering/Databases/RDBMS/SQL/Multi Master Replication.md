# Multi Master Replication

- This technique allows multiple database servers to act as both master and slave, which means that changes made to any of the servers are automatically propagated to all the other servers.
- Multi-master replication in MySQL is based on the concept of a circular replication topology, where each server acts as both a master and a slave to the other servers in the topology.
- This means that any server in the topology can accept updates and replicate them to the other servers, while also receiving updates from the other servers and replicating them locally.

To set up multi-master replication in MySQL, you need to perform the following steps:

1. Configure each MySQL server to use the same replication user and password, and make sure that the servers can communicate with each other over the network.
2. Set up each MySQL server as a master by enabling binary logging and creating a unique server ID for each server.
3. Configure each MySQL server as a slave by specifying the master server's hostname, port number, replication user, and password.
4. Enable circular replication by configuring each MySQL server to replicate changes from the other servers in the topology.
5. Monitor the replication topology to ensure that changes are being propagated correctly and that there are no conflicts or inconsistencies.

## Advantages

- It provides a way to distribute the workload across multiple servers and increase the availability and scalability of the database system. It also allows for read and write operations to be performed on any server in the topology, which can improve performance and reduce latency.

## Disadvantages

- Main challenge is managing conflicts that can arise when multiple servers update the same data concurrently. To avoid conflicts, you need to implement some form of conflict resolution mechanism, such as using timestamps or unique identifiers to ensure that conflicting updates are resolved in a consistent manner.
- Another challenge is ensuring that the replication topology remains consistent and reliable, especially in the face of network disruptions or hardware failures. To address these issues, you need to implement robust monitoring and failover mechanisms that can detect and recover from failures quickly and efficiently.