# The Dual Write Anti-Pattern

Take this example; consider there is a system that handles orders for a users on a shopping website. 

Here, order service gets request to initiate the order first, and this must then propagate throughout the system.

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled.png)

In general, this is what something like this can look like.

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%201.png)

An event to save something to the database happens simultaneously as a follow-up event takes place; this could be to update Redis or something else.

What happens if the application fails after the save has been completed but before the emit finishes? Our database was updated in this scenario, but our event was never successfully emitted. This is a problem because we have a huge inconsistency between our database and our events.

# Transaction Doesn’t Solve

One solution is wrapping everything under a transaction; such that if a failure occurs, transaction gets rolled back, and DB change is reverted.

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%202.png)

But what happens if failure occurs after event has been emitted? Then event has already been emitted but DB changes will get rolled back.

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%203.png)

# The Four Solutions

## **Event Chaining, CDC and Deduplication**

Chain events and relying on deduplication.

By making the emitting of events dependent on the success of the database action, we’re ensuring an event has only occurred once the previous step is successful.

Add an async process that reads DB for changes and pushes them to MQ.

You may think:

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%204.png)

> Yes, but what if the app still breaks before an event is fully emitted?
> 

Good question; as we’ve decided to chain everything off success states, we can now rely on our event system to handle the deduplication of events, which hopefully you have in place (as all good event systems should). It should look something like this:

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%205.png)

A CDC pattern can be used to do this async processing robustly

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%206.png)

### Deduplication

[https://medium.com/geekculture/efficient-event-streamlining-and-dynamic-de-duplication-across-message-brokers-81911f47264a](https://medium.com/geekculture/efficient-event-streamlining-and-dynamic-de-duplication-across-message-brokers-81911f47264a)

This technically doesn’t solve the problem entirely, as it doesn’t prevent the dual write problem; it merely handles issues that may occur because of it. This should, in theory, resolve your issues in the long term, but as I mentioned above, it’s rather inelegant.

So, what happens if we don’t have event deduplication in place or don’t want to rely on this to solve our problem?

## **The Outbox Pattern**

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%207.png)

- In this pattern, our app uses a transaction to update the state and the outbox table at the same time.
- The outbox is a log of the events that need to be emitted.
- A seperate process can scan the outbox and emit any new events.

## Event Sourcing Pattern

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%208.png)

- Event sourcing eliminates the storage of state and only stores the events.
- We then run a process against the event log and emit anything new.
- Meanwhile the event log can be used to reconstruct the state when required.

## Listen To Yourself Pattern

![Untitled](The%20Dual%20Write%20Anti-Pattern/Untitled%209.png)

- Reverses all of this and emits the events first.
- A seperate process then listens to these events and updates the state.