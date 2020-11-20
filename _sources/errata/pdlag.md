# Phase Detection Lag

_Phase detection lag_ occurs when a a component is notified of a phase change one interval after the phase has actually ended. This occurs because the phase detector is unable to report on the current phase until the interval is ended so it can generate a signature and classify it. This is an issue for model swapping: although we can hold off on training models until the end of a phase when we know what phase we were in, we cannot do the same when a model is already swapped. This means that when we are running with a swapped model, the interval after the end of a phase will be run with a model trained for the phase that just ended, instead of the proper model. 

```{figure} ../images/pdlag.png
---
height: 225px
name: pdlag-fig
---
A visual depiction of phase detection lag.
```

## Potential Solutions

### Checkpoint-Restart
We can checkpoint the simulation at the end of every interval, and rollback if we find out that we ran with the wrong model. In a simulation environment with an intelligent checkpoint restart mechanism, we may be able 
to only redo computation that depended on components that were rolled back. However, I don't believe SST supports this, and and I think that even with this in place, the overhead would be large for simulations with short phases. We would have to rollback everytime a phase ended, essentially. 

### Phase prediction
Phase prediction involves predicting what the next interval will be classified as before it is run. This can be thought of as two separate problems: (1) how long will the current phase run for, and (2) what will be the next phase that runs. There are two ways this could help alleveiate the problem:
* We could implement phase prediction on its own. With a high enough percentage of correct predictions, we would not be running the wrong model as frequently, which would improve the accuracy of our simulation. 
* We could impelement phase prediction in addition to checkpoint restart. Correct predictions would mean that we would alleviate the need for a rollback on correctly predicted phase endings. It does not alleviate the need to checkpoint at every interval though, as we may mispredict a phase ending and still need to rollback.

### Run-ahead Phase Detection
We could move the phase prediction earlier in the simulation, and have it run one interval ahead of the rest of the simulation. This would mean that the phase detector could send perfect phase information to listeners. 

This is not a silver bullet though. Drawbacks include:
* If the phase detector is run before the simulation of an interval, it will only have access to data from the program trace it is reading. Some approaches to phase detection using things like branch mispredictions as part of their classification criteria, which would not be available to us. 
* Speaking of program traces, this approach will hurt the generality of our implementation, as it will assume that it has access to a program trace, which is not true if the core is simulating the code itself, as opposed to replaying a trace it is given. 
* Finally, we can't forget that online phase detection has application in hardware. If we want our approach to work in this area, it can't have magic knowledge of the future. 
