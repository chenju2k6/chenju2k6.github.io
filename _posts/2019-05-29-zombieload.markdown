---
layout: post
title:  "What is ZombieLoad and how it is related to Meltdown-type attacks"
date:   2019-05-29 23:00:28 -0500
categories: jekyll update
---
有的时候我真的怀疑这些漏洞是不是精心设计的后门呢？

Anyway, following the discovery of Meltdown and Spectre in early 2018, this year researchers found a series of attacks exploits the vulnerabilities in the microarchitecture design of the Intel CPU, they are ZombieLoad, Fallout, RIDL and others. I would like to thank the people who draw this excellet interative map illustrating the internal CPU microarchitecture. ![alt text](https://mdsattacks.com/images/skylake-color.svg "Microarchitecure") The interactive map can be found in [here](https://mdsattacks.com/diagram.html). This drawing helps a lot to compare and contrast these attacks.

## Out-of-order execution and Tomasulo algorithm

As suggested by the name, the out-of-order execution allows the instructions to be executed in the order different from the original one defined in the program stream, while still preserving the program's sementaic. The purpose of doing this is to maximize the utilization of the CPU execution units and increase the processing throughput. The idea is that if an execution unit is occupied and stalls the pipeline, the following instructions can be scheduled to utilize the free execution units. 

The Tomasulo algorithm is designed to schedule the instructions to allow out-of-order execution. The algorithm uses an unified researvation stations (RS). Instructions will first be issued and queued in the RS. If all the operands are ready, the instruction will be dispatched to the instruction unit to run. There is a bus, called common data bus to connet RS and all the execution units. The RS will listend on the CDB for the requiring operands. The RS will also rename the registers names and put it to a register alias table (RAT) to avoid RAW, WAR and WAW hazards. Consider the below example

```
F1 = F2 + F3
F4 = F1 - F2
F1 = F2 / F3
F2 = F4 + F1
```
