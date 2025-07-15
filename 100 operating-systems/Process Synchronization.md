
## **Process Synchronization

When multiple processes or threads need to interact with each other, they often do so by **communicating** or **competing** for shared resources. A common way for processes to communicate is by using **shared memory**. However, this introduces a significant challenge.

> I would assume this thing is for the sake of simplicity

So, there is a shared memory, which as the name suggests is a memory shared by all processes, this is said explicitly, cause we discussed before that every process has its own, resources, and information, but if processes want to communicate with each other, and basically want to work on the same data, the data needs to be public, that is basically in the shared memory (what about other resources, how does that, work, like is everything that is supposed to be shared brough to the ram, i have to check this out)

Now, 