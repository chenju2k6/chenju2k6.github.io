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
F2 = F4 + F1
F1 = F2 / F3
F4 = F1 - F2
F1 = F2 + F3
```

The fist instruction will be put to the reservation station for the ADD unit. Because the result is not ready, which means the F2 can not contain the latest result, the RAT table will have a entry with F2 points to RS1, meaning the latest value of F2 will be updated once the RS1 is finished executing. The next instruction, will be queued to the free reservation station entry, RS2. Meanwhile, because the operation depends on F2, one of its operand wil be marked as RS1, and there will be a new entry in the RAT with F1 points to RS2. Similarly, the next instruction will be inserted to RS3 with its two operands marked as RS1 and RS2. The last instruction will be inserted to RS4 with its operands marked as F3 and RS1. Interestingly, this operation will also replace the orginal entry ("F1 points to RS2") with ("F1 points to RS4"). This is because the latest result will be depends on the execution result of the last execution which is placed in RS4. Noted that the original dependency relationship is not violated, because once RS2 finishes executing, the RS3 will be notified. The entry with replaced content will only affect the upcoming instrutions which are not issued yet. 

There is a great [resource](https://classroom.udacity.com/courses/ud007) to learn Tomasulo algorithm.

In Tomasulo algorithm, the instructions are issued by the program order, but not necessariliy dispatched by the program order. In addition, for load and store instructions, the Tomasulo algorithm follows the order defined by the original program.

## Implementation of out-of-order execution 

On Intel CPUs, the out-of-order is implemented by employing the following components, reorder buffer for register allocation, register naming and retiring. The unified revervation station connects the execution units and performs like a schedulear. The execution units include ALU, AGU and load/store units. The load and store connects to the memory subsystem. Intel CPUs also implement speculative execution which means the instructions on one branch path that are predicted by the branch prediction unit will be executed directly. If the prediction is wrong, then the reorder buffer will be cleared and the unified reservation station will be re-initialized.


## Exploit of the microarchitecture

|Attach type|Components explots|sampling|
|---|:---:|---:|
|Meltdown|TODO|No|
|Spetre|TODO|No|
