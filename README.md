# PseudoCode-for-Starve-free-Reader-Writer-Problem
## Description
Let us first understand what this problem is before diving into the pseudocode. The readers-writers problem is a well-known challenge in computer science. It involves a shared resource, such as a database, that can be accessed by readers, who only need to read the resource, and writers, who can make modifications to it. While a writer is making changes to the resource, no other reader or writer should be allowed to access it simultaneously since if multiple writers were to access the resource at the same time, the resource could become corrupted. Similarly, if a reader were to read a partially modified value, the value could be inconsistent and lead to errors.
## Variations
There are three different variations to this problem solution approach
* Readers have priority (First Readers Writers Problem) : 
If there are one or more readers currently accessing the resource, new readers should also be allowed to access it. However, this approach may lead to starvation if there are writers waiting to modify the resource and new readers keep arriving. In such cases, the writers will never be able to access the resource as long as there is at least one active reader. In this case Writers starve.
* Writers have priority (Second Readers Writers Problem): 
In this approach, writers are given priority over readers. However, this can cause readers to starve, as they may have to wait for a long time before being granted access to the resource. In this case Readers starve.
* No priority given (Third Readers Writers Problem): 
This approach grants access to the resource in the order of arrival for both readers and writers. If there are readers already accessing the resource, a writer will have to wait until they finish before being able to modify the resource. Similarly, new readers arriving while others are accessing the resource will have to wait until the resource is free. We're using fair queuing between readers and writers in order to have a starve free solution.
## PseudoCode:
* Step 1 (Basic structure): 
In order to prevent starvation and ensure that both readers and writers have fair access to the resource, we will use a semaphore called "orderMutex" to represent the order of arrival. This semaphore will be acquired by any entity that requests access to the resource and will be released as soon as that entity gains access to the resource. By using the orderMutex semaphore in this way, we can ensure that all entities accessing the resource are queued fairly, regardless of whether they are readers or writers. This will help to prevent any one entity from monopolizing the resource and will ensure that all entities have a fair chance of accessing it.
```
semaphore orderMutex=1;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the writer can access the resource
}
```
* Step 2 (Writer Code): 
To ensure that the writer has exclusive access to the resource while modifying it, we will create a new semaphore called "accessMutex". The writer will be required to acquire this semaphore before making any modifications to the resource. This will prevent other entities, including readers and writers, from accessing the resource while the writer is modifying it.
```
semaphore accessMutex=1;     // Initialized to 1
semaphore orderMutex=1;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  
  P(accessMutex);          // Request exclusive access to the resource
  
  V(orderMutex);           // Release order of arrival semaphore (we have been served)

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
}
```
* Step 3 (Readers Code): 
For the Readers we know multiple readers can access the resource simultaneously. To prevent writers from accessing the resource while a reader is using it, the first reader to access the resource should lock it. Similarly, when a reader is finished with the resource, it should release the lock on the resource if no other readers are currently using it. To implement this, we will use a counter called "readers" to keep track of the number of readers currently accessing the resource, as well as a semaphore called "readersMutex" to protect the counter against conflicting accesses. By using these tools, we can ensure that the resource is accessed safely by multiple readers without causing conflicts with any writers who may also need to access the resource.
```
semaphore accessMutex=1;     // Initialized to 1
semaphore readersMutex=1;    // Initialized to 1
semaphore orderMutex=1;      // Initialized to 1

unsigned int readers = 0;  // Number of readers accessing the resource

void reader()
{
  P(orderMutex);           // Remember our order of arrival

  P(readersMutex);         // We will manipulate the readers counter
  
  if (readers == 0)        // If there are currently no readers (we came first)...
    P(accessMutex);        // ...requests exclusive access to the resource for readers
  readers++;               // Note that there is now one more reader
  
  V(orderMutex);           // Release order of arrival semaphore (we have been served)
  V(readersMutex);         // We are done accessing the number of readers for now

  ReadResource();          // Here the reader can read the resource at will

  P(readersMutex);         // We will manipulate the readers counter
  
  readers--;               // We are leaving, there is one less reader
  if (readers == 0)        // If there are no more readers currently reading...
    V(accessMutex);        // ...release exclusive access to the resource
  
  V(readersMutex);         // We are done accessing the number of readers for now
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  
  P(accessMutex);          // Request exclusive access to the resource
  
  V(orderMutex);           // Release order of arrival semaphore (we have been served)

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
}
```
## Summary and Conclusion:
* This solution uses three semaphores: accessMutex, readersMutex, and orderMutex. Mutual Exclusion is handled here. accessMutex is used to ensure exclusive access to the resource for writers and also to coordinate exclusive access among readers. readersMutex is used to ensure exclusive access to the readers counter, which keeps track of how many readers are currently accessing the resource. orderMutex is used to preserve the order of arrival of readers and writers and prevent starvation.
* In the reader function, readers acquire the orderMutex semaphore first to remember their order of arrival. Then, they acquire the readersMutex semaphore to access the readers counter and increment it. If the reader is the first one to arrive (i.e., readers == 0), the reader acquires the accessMutex semaphore to obtain exclusive access to the resource for readers. Once readers finish accessing the resource, they decrement the readers counter and release the accessMutex semaphore if they were the last reader to leave.
* In the writer function, writers acquire the orderMutex semaphore first to remember their order of arrival. Then, they acquire the accessMutex semaphore to obtain exclusive access to the resource for writers. Once the writer finishes modifying the resource, it releases the accessMutex semaphore.
* The systems cannot enter a state of deadlock. As long as the time of execution is finite for all the processes in their critical sections, there will be progress as the processes will keep on executing.
* Overall, this solution ensures that readers and writers get access to the resource in the order they arrive and without causing any starvation. Therefore, the solution is starvation-free for both readers and writers.
