Having concluded the course of my studies, I decided to gather the most significant reports I've written and presented for my assignments and make them available here in case anyone wants to read them, in an effort to save them from oblivion. Here goes nothing!

## Master Thesis · *A control system with Watchdogs and Data Logging Capabilities for the USAD vehicle*

## Bachelor's Degree Final Report · *Linux as a Real-Time Operating System. A practical analysis in the Mobile Robotics domain*

## Computer Security · *Speculative Execution Side Channel Attacks* 

In recent times there's been much discussion about mitigations of CPU vulnerabilities, with some people stating that disabling them is an incredible security risk and that the performance implications are negligible anyway, while others claim that the opposite is true.

In an effort to understand the impact of disabling mitigations, I decided to investigate Transient Execution Side Channels attacks and write a little article for this blog. This investigation, however, turned into a much more fleshed out and in-depth analysis of Spectre attacks which I presented as a final assignment for the course of *Computer Security*.

Writing this report was really interesting, as it allowed me to tie together very different aspects of computing, and get a better understanding of how the complex speculative dispatch and execution units in modern CPUs work.

## Complex Systems: Models and Simulations · *Smoke Propagation Simulator*

> This report is in Italian

For the course of *Complex Systems: Models and Simulations* I wrote a simulation of smoke propagation in a 2D indoor environment using Cellular Automatas; the program was written in C(++) using the Dear ImGui library for visualization. This work was somewhat rushed, so my approach wasn't all that novel, but the visualization tool turned up nicely and allowed me to test several propagation methods.

The algorithm is based on a pre-existing paper that simulated a 3D environment, and was scaled down to two dimensions. The simulation is neither accurate nor granular enough for accurately reproducing the flow of smoke in a building, but it's incredibly fast and uses very limited resources compared to traditional CFD (Computational Fluidodynamics, i.e. insanely complex mathematics with calculations performed offline in Fortran) and could potentially run on a microcontroller to preemptively trigger alarms and/or other devices along the predicted path of smoke in case of fire in an enclosed environment.

## Parallel Computing · *The Riken K Computer Architecture* and *Performance Analysis of the GFC Compression Algorithm*

During the *Parallel Computing* course we'd been given two assignments: hold a seminary on a topic of your choice and analyze one algorithm and implement it in CUDA, MPI and/or OpenMP.

For the seminary, I decided to analyze the architecture of the Riken K Computer, which had once been the fastest supercomputer in TOP500. Alas, the recording of the seminary has never surfaced, but the slides are still available: for the time being, I'll upload them here; I hope I'll be able to recover the recording in the future and turn it into an article for the blog.

As for the analysis and implementation assignment, I chose the GFC compression algorithm, which was written explicitly to exploit the data parallelism of GPUs. Since a CUDA implementation was already available, I dissected it and performed an in-depth performance analysis which digressed into an interesting CUDA SASS disassembly analysis. I then wrote a reference implementation of the algorithm in OpenMP, which is obviously not as efficient due to different memory models between CPUs and GPUs.

<!--
## Analysis of the "CouchDB" database

Another very interesting report I wrote is an analysis of how the CouchDB non-relational database works internally.

I won't be sharing this work for the time being, as I wasn't the sole author. 
-->