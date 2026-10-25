# Race Condition vs. Data Race

[https://blog.regehr.org/archives/490](https://blog.regehr.org/archives/490)

A race condition is a non-deterministic; temporal event. A data race is something that is a property of the code; you can look at the code and spot data races.

## Race Condition

When the timing or ordering of events affects a program’s correctness. Some external timing or ordering non-determinism is needed to produce a race condition; Eg. context switches, OS signals, memory operations on a multiprocessor, and hardware interrupts.

## Data Race

When there are two memory accesses in a program where:

- both target the same location
- both are performed concurrently by two threads
- atleast one of them is a write
- both are not synchronization operations

“*Concurrently*” means there wasn’t anything like a lock that forced one operation to happen before or after the other. 

“*Are not synchronization operations*” there are special memory operations, such as those used to implement locks, that are not themselves synchronized. By definition, these do not trigger data races.

## Relationship Between the Two

These can occur together, and also independently of each other.

### Race Condition due to Data Race

```java
transfer1 (amount, account_from, account_to) {
  if (account_from.balance < amount) 
		return NOPE;
  account_to.balance += amount;
  account_from.balance -= amount;
  return YEP;
}
```

When called from multiple threads without external synchronization, this function admits both data races (multiple threads can concurrently try to update an account balance) and race conditions (in a parallel context it will create/lose money out-of/into thin air). 

### Race Condition without Data Race

```java
transfer2 (amount, account_from, account_to) {
  atomic {
    bal = account_from.balance;
  }
  if (bal < amount) return NOPE;
  atomic {
    account_to.balance += amount;
  }
  atomic {
    account_from.balance -= amount;
  }
  return YEP;
}
```

Assume “atomic” is implemented simply by acquiring a thread mutex at the start of the atomic block and releasing it at the end, perhaps using some sort of transaction, or perhaps by disabling interrupts; ensuring the code inside the block executes atomically.

The problem with transfer2 is that it permits other threads to see memory states where a key invariant — conservation of money — is broken.

### No Race Condition and no Data Race

```java
transfer3 (amount, account_from, account_to) {
  atomic {
    if (account_from.balance < amount) return NOPE;
    account_to.balance += amount;
    account_from.balance -= amount;
    return YEP;
  }
}
```

### No Race Condition but Data Race

```java
transfer4 (amount, account_from, account_to) {
  account_from.activity = true;
  account_to.activity = true;
  atomic {
    if (account_from.balance < amount) return NOPE;
    account_to.balance += amount;
    account_from.balance -= amount;
    return YEP;
  }
}
```

Here we are setting flags to indicate activity on the accounts; perhaps for auditing. Data Races on these flags are not harmful, hence no race condition here. 

## Conclusion

We’ve ended up covering all possibilities:

|  | data race | no data race |
| --- | --- | --- |
| race condition | transfer1 | transfer2 |
| no race condition | transfer4 | transfer3 |

### Limitations of Identifying Data Races

t2 and t4 show that Lack of data races is neither necessary nor sufficient for establishing the concurrency correctness of a computer program. 

### Uses of Identifying Data Races

1. A data-race-free program is majorly independent of the whims of whatever weak memory model the language sits on. 
2. Data races are undefined behavior in C and C++ and tend to have complicated semantics in other languages. 
3. Data races are easy to find automatically — notice from the definition that it simply requires memory accesses to be monitored; no knowledge of application semantics is required. 

Race conditions, on the other hand, are intimately tied to application-level invariants, making them far thornier to reason about (quick — what are all the invariants in Firefox? In Linux? In MySQL?).