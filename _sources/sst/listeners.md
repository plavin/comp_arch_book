Cache Listeners
======================

SST caches allow objects to register as listeners, so that they recieve information on what the cache is doing. Currently, listeners are things like profilers, analyzers, and prefetchers. We should check out what that interface looks like. It is specified in cacheListener.h [[source](https://github.com/sstsimulator/sst-elements/blob/master/src/sst/elements/memHierarchy/cacheListener.h)].

Here is the information contained in CacheListenerNotification, which is the type of object sent from Caches to CacheListeners:

```{code-block} c
    uint32_t size;
    Addr targAddr;
    Addr physAddr;
    Addr virtAddr;
    Addr instPtr;
    NotifyAccessType access;
    NotifyResultType result;
```

As you can see, this is a rather limited amount of information, but it is enough for basic phase analysis, which sometimes requires only the instruction pointer. 

Questions: 

1. So when are these generated? 
2. If we register as a listner to the L1 cache, will we be informed of every cache access? 
3. What about prefetches, are those included?
4. Can we register multiple listeners?

memoryController.cc [[source](https://github.com/sstsimulator/sst-elements/blob/master/src/sst/elements/memHierarchy/memoryController.cc)] should have the answers.

Answers:

1. When the memory controller recieves an event, it calls `notifyListeners`. This is defined in memoryController.h, and simply generates an object of the type seen above. Each listner receives this. The last two arguments are always READ and HIT, respectively. It achieves this by creating the notification object, then sends it using a callback.
```{code-block} cpp
void notifyListeners( MemEvent* ev ) {
        if (  ! listeners_.empty()) {
            // AFR: should this pass the base Addr?
            CacheListenerNotification notify(ev->getAddr(), ev->getAddr(), ev->getVirtualAddress(),
                        ev->getInstructionPointer(), ev->getSize(), READ, HIT);

            for (unsigned long int i = 0; i < listeners_.size(); ++i) {
                listeners_[i]->notifyAccess(notify);
            }
        }
    }
```
1. The listener is attached to the memory controller, not the cache.
2. Prefetches are probably included, as every event received by the controller is sent to listeners.
3. Yes. Not sure about the limit though.

More questions: 
1. How do we send information back to the component? Answer: A callback is registered when the listener is created. The cassini strideprefetcher, for instance, uses this to send new memory events back to the controller, which are prefetch events. 
2. How can we use this? Answer: We'll need to look into sending custom events, I think. TODO. 


