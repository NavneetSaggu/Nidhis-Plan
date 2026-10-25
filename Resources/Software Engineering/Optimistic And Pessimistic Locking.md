# Optimistic And Pessimistic Locking

When dealing with conflicts, you have two options:

- You can try to avoid the conflict, and that's what **Pessimistic Locking** does.
- Or, you could allow the conflict to occur, but you need to detect it upon committing your transactions, and that's what **Optimistic Locking** does.

# Optimistic Locking

Strategy where you read a record, take note of a version number (other methods to do this involve dates, timestamps or checksums/hashes) and check that the version hasn't changed before you write the record back. When you write the record back you filter the update on the version to make sure it's atomic. (i.e. hasn't been updated between when you check the version and write the record to the disk) and update the version in one hit.

If the record is dirty (i.e. different version to yours) you abort the transaction and the user can re-start it.

This strategy is most applicable to high-volume systems and three-tier architectures where you do not necessarily maintain a connection to the database for your session. In this situation the client cannot actually maintain database locks as the connections are taken from a pool and you may not be using the same connection from one access to the next.

# Pessimistic Locking

When you lock the record for your exclusive use until you have finished with it. It has much better integrity than optimistic locking but requires you to be careful with your application design to avoid [Deadlocks](http://en.wikipedia.org/wiki/Deadlock). To use pessimistic locking you need either a direct connection to the database (as would typically be the case in a [two tier client server](http://en.wikipedia.org/wiki/Client-server) application) or an externally available transaction ID that can be used independently of the connection.

In the latter case you open the transaction with the TxID and then reconnect using that ID. The DBMS maintains the locks and allows you to pick the session back up through the TxID. This is how distributed transactions using two-phase commit protocols (such as [XA](http://www.opengroup.org/bookstore/catalog/c193.htm) or [COM+ Transactions](http://msdn.microsoft.com/en-us/library/ms687120(VS.85).aspx)) work.