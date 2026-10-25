# Locking Implementation

We require a mechanism to manage the locking requests made by transactions. Such a mechanism is called a **Lock Manager**

# Data Structure in Lock Manager

The data structure required for the implementation of locking is called a Lock Table

1. It’s a hash table, keys of data items are hashed to one entry in the table
2. Each locked data item has an associated linked list.
3. Every node in the linked list is a transaction that requested for lock.
4. Each lock has the mode of lock requested (mutual/exclusive), and the current status of the request (granted/waiting).
5. New lock requests are added to the end of the linked list.
6. Collisions are handled by separate chaining.

![Untitled](Locking%20Implementation/Untitled.png)

- Here, 5, 47, 167 and 15 are data items
- 5 and 47 collide to the same Lock Table entry

# Working of Lock Table and Lock Manager

- Initially, the lock table is empty as no data item is locked.
- Whenever the lock manager receives a lock request from a transaction T on a particular data item Q following cases may arise:
    - If Q is not already locked, a linked list will be created and a lock will be granted to the requesting transaction T.
    - If the data item is already locked, a new node will be added at the end of its linked list containing the information about the request made by T.
- If the lock mode requested by T is compatible with the lock mode of the transaction currently having the lock, T will acquire the lock too and the status will be changed to ‘granted’. Else, the status of T’s safety will be ‘waiting’.
- If a transaction T wants to unlock the data item it is currently holding, it will send an unlock request to the lock manager. The lock manager will delete T’s node from this linked list. The lock will be granted to the next transaction in the list.
- Sometimes transaction T may have to be aborted. In such a case all the waiting requests made by T will be deleted from the linked lists present in the lock table. Once abortion is complete, locks held by T will also be released.