MemEvent objects
======================

SST's MemHierarchy element implements objects like caches and memory controllers. The interface for these objects are the MemEvent and MemEventBase objects. It is useful to know what these are, but they are not part of the interface to cache listeners, which is an important distinction. So, while it is possible to get things like the addresses in the listeners, it is not possible to get the prefetch bool. The interface to CacheListener is described in the next chapter.

If a phase analysis algorithm arises that needs information not in that interface, we will need to talk to SST about changing the interface to CacheListener. 

The memEventBase object [[source](https://github.com/sstsimulator/sst-elements/blob/master/src/sst/elements/memHierarchy/memEventBase.h)] contains the following fields:

```{code-block} c
    id_type         eventID_;           // Unique ID for this event
    id_type         responseToID_;      // For responses, holds the ID to which this event matches
    string          src_;               // Source ID
    string          dst_;               // Destination ID
    string          rqstr_;             // Cache that originated this request
    Command         cmd_;               // Command
    uint32_t        flags_;
    uint32_t        memFlags_;
```

As you can see, this does not contain the information needed by most phase detection algorithms, 
such as the address we want to read from or write to! For this, we will need the memEvent object [[source](https://github.com/sstsimulator/sst-elements/blob/master/src/sst/elements/memHierarchy/memEventBase.h)], which extends memEventBase with the following fields:

```{code-block} c
    uint32_t        size_;              // Size in bytes that are being requested
    Addr            addr_;              // Address
    Addr            baseAddr_;          // Base (line) address
    bool            addrGlobal_;        // Whether address is a local or global address
    MemEvent*       NACKedEvent_;       // For a NACK, pointer to the NACKed event
    int             retries_;           // For NACKed events, how many times a retry has been sent
    dataVec         payload_;           // Data
    bool            prefetch_;          // Whether this request came from a prefetcher
    bool            blocked_;           // Whether this request blocked for another pending request (for profiling) TODO move to mshrs
    bool            dirty_;             // For a replacement, whether the data is dirty or not
    bool            isEvict_;           // Whether an event is an eviction
    Addr            instPtr_;           // Instruction pointer associated with the request
    Addr            vAddr_;             // Virtual address associated with the request
    bool            inProgress_;        // Whether this request is currently being handled, if in MSHR TODO move to mshrs
```

Alright, now we see the information we may need to better do phase ananlysis. In particular, the fields `addr_`, `baseAddr_`, and `instrPtr_`. Ok, so we know we'll want to take a look at these objects as they move through the memory hierarchy. How will we get access to them? 

SST caches allow objects to register as listeners, so that they recieve information on what the cache is doing. We should check out what that interface looks like. It is specified in cacheListener.h
