# Background
For this master project, our initial goal was to find ways to improve runtime energy consumption of a program with the help of compiler optimizations.

The idea was to find instructions that would reduce energy consumption without changing the result of the program.

In our [initial presentation](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/resources/Initial%20Presentation.pdf), we discussed the Related work on this topic, which led us to defining our new goal for this master project.

For each of the papers, there should exist 3 files:
1. .pdf - The paper itself
2. .txt - A pdftotext conversion of the paper, good for searchability
3. .md - Our notes&thoughts regarding the paper *(does not exist for all papers)*

A fundamental concept in runtime energy-efficiency is the classification of tasks as either memory-bound, or compute-bound.
Memory-bound tasks benefit from reduced CPU clock frequency, as the CPU is stalled for memory most of the time and additional cycles do not benefit the overall performance (Known as crawl-to-sleep).
For CPU-bound workloads however it is usually best to run at high clock frequency (and power demand) to finish the workload faster and reach a low-power state (Known as Run-To-Sleep). This is most efficient as usually the static power demand caused by running for a longer time is greater than the increased dynamic power demand.
![Energy-Efficiency-Fundamentals](uploads/f03d01ba5fcc09337051337e2865cd91/Energy-Efficiency-Fundamentals.png)

# Related Work
In this section, we highlight some of the most important research resources we used to gain an understanding of the problem domain and to define our goals and expectations.

We follow the same path of research papers here as we did in our initial presentation.
![Related-Work](uploads/b98ff82652c28c3858568f28eb57e031/Related-Work.png)

For a complete list of research papers & our notes, see the [literature folder](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/tree/master/literature) in the main repository.

## Processor Cruise Control - *Bellosa et al. 2002*
This paper outlines a process that uses processor performance counters to identify how memory-bound a process is and adjust the dynamic voltage and frequency scaling (DVFS) accordingly.

For memory-bound workloads this results in only small decreases in performance, whilst achieving significant power savings.

As this paper is from 2002, its relevance for modern processors can be questioned, as the paper only tests the behavior on a single-core processor. Furthermore, the behavior of a modern processor may be substantially different. The number of processors, microarchitectures, as well as fabrication processes have changed significantly since 2002.

However, it backs up the initial assumption that memory-bound workloads benefit from reduced clock frequency.

## Compile-time dynamic voltage scaling settings - *Xie et al. 2003*
"Processor Cruise Control" used reactive DVFS to achieve reduced runtime energy consumption.
This leaves the question whether this can also be achieved pro-actively, i.e. by analyzing the project at compile-time and inserting the appropriate frequency-scaling instructions.

Xie et al. work to answer this question in this paper.
They use two different methods of estimating the power savings achievable by inserting different mode set instructions.
The first method is an analytical model for the upper bound of possible savings, which does not consider penalties for switching power modes.
Second, they use a MILP (mixed integer linear program) based optimization, which heeds penalties of switching frequencies.

The target of their paper are multimedia applications (i.e. audio processing), where deadlines need to be met, but achieving the deadline early doesn't yield any benefit.

Their approach seems to result in good power savings (up to a factor of 2x).
However, much of the paper is spent discussing how to achieve a good result in cases where the ideal frequency cannot be set exactly, but lies in between two choose-able frequency/power options.
It seems the result of their work in this regard is that continually switching between the two frequency/power modes that are closest to the ideal, achieves close to ideal results.

An open question here is how many voltage/frequency pairs are available on a modern CPU and whether the approach of switching between them still leads to closer to ideal results.