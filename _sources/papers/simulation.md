Simulation
====================

## Mocktails

**Title: Mocktails**: Capturing the Memory Behavior of Proprietary Mobile Applications

**First Author**: Mario Badr

**Link**: [IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/9138919)

### Overview 

While new heterogeneous hardware is being created all the time, it is uncommon for companies to release memory traces for proprietary hardware. The authors introduce Mocktails, which aims to "synthetically recreate spatio-temporal memory access behaviour of proprietary heterogeneous compute devices." Mocktails is able to generate a statistical profile from a proprietary trace, which it can then turn into a synthetic trace which retains the statistical properties of the original.

This interests us because we are interested in the automatic generation of memory benchmarks, which must also make an attempt to be hardware agnostic after generation. We are also interested in modeling memory behavior and as such we should investigate what statistics they needed to capture to be able to accurately produce a synthetic trace.
