Model Size
=====================

Model size must be carefullly accounted for in the analysis of our work. While a single phase can be represented by a simple model, we have a separate model for each phase, and not only that, we initially train multiple models for each phase. This means that unless our models are substantially smaller, we will probably end up using more space for our models that the base model would use.

So, is this a problem? I do not fully understand how much memory a single rank of SST currenly uses. If it is does not need much state, then it may be ok to increase the state size by a small constant factor. But this is certainly something we need to both track, and present in our final work. 

## Potential Approaches

1. We may find that the trade-off is worth it: if our statistical models are substantially faster, the memory trade off may be worth it.
1. If we find the trade-off is not worth it, we must choose simpler models, or train fewer models. 
