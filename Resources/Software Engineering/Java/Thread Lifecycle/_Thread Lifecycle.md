# Thread Lifecycle

![Untitled](Thread%20Lifecycle/Untitled.png)

- Runnable means that thread is in the executable and it is doing something. This indicates CPU usage.
- Timed Waiting means you’re sleeping, and you’re waiting for the interrupt to hit when your sleep will be over. Or you’ve started another thread and you’re waiting for that thread to finish. It’s doing nothing, it’s sittingm waiting for something to happen. This is usually not an issue.
- Blocked means there’s a synchronization block, your thread is waiting on a resource. This is usually a probable cause for contention. These could also be IO hoggers, (remember how sync logging was slowing down EDE and CM?)
- Terminated means it’s finished.

These are the main four relevant thread states.