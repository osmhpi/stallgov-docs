For the development of Memutil, we are testing & evaluating different heuristics, to achieve the optimal performance/Watt. The different heuristics we came up with are explained here.

# General problem
Figure out how memory-bound a workload is.
Complication: high memory-stalls != memory-bound -> memory unit might be stalled, but enough other instructions are available to execute. Furthermore, the memory that is stalled on might be the L1 or L2 cache, which is in the same clock domain as the core, therefore benefiting from increased clock frequency.
See the "Evaluation" section of the [Intel Counters](Counters-Intel) page for more information.

# Current best heuristic: frequency interpolation on the l2stalls/cycle (Intel)
As explained on the [Counters](Counters-Intel) page, our evaluation of different performance counters suggested a shift in our categorization from memory-bound vs. cpu-bound toward off-core-bound vs. on-core-bound. The L1 and L2 cache "memory" is part of the on-core frequency domain of each core. Therefore benefiting from increased clock frequencies and part of on-core-bound workloads.

In commit cf29e03a we introduced a CpuFreq Governor based on linearly interpolating between the minimum and maximum frequencies, depending on the ratio of l2stalls/cycle.
Where 65% l2-miss-stalls/cycle would result in the minimum clock frequency and 10% l2-miss-stalls/cycle would result in the maximum frequency.
These values are tuned for a laptop with a core i7 6700HQ and will most likely need to be tweaked for different machines.

Our evaluation for this approach looks very promising, as this heuristic provided benefit in all off-core-bound workloads (mg.C, cg.B, ft.B).
The other workloads are not off-core-bound and therefore don't benefit from frequency adjustments. (apart from turbo boost which we currently cannot control)

Therefore we believe that, for now, focusing on improving the behavior of our governor based on this heuristic is the best approach - at least for Intel processors.

Power/Runtime evaluation:
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

Memutil log:
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/log-core0.png)

Further research is needed on the behavior of the l2stalls metric at different frequencies, the best behavior at idle, as well as the effects of switching the frequency quickly/radically.

# Current best portable heuristic: frequency interpolation on the instructions/cycle
Unfortunately we require a very specific counter for the heuristic given above. As this counter
is not always available (e.g. if an AMD CPU is used) we tried to find a heuristic that uses more generally available counters. The IPC (instructions per cycle) heuristic is the result. The plots on the [Counters Intel](Counters-Intel) and [Counters AMD](Counters-AMD) page show that our workloads that benefit from higher frequency have an IPC of more than 0.4 while workloads that need a lower frequency have a lower IPC like 0.25 for the mg.C workload.

In commit `86ba140e9122f0cae7044a934776dae0d3d922cb` we introduced a heuristic that uses a linear interpolation between the maximum and minimum available frequency depending on the IPC. For an IPC of equal or greater than 0.45 we use the maximum available frequency while for an IPC of 0.1 or less we use the minimum available frequency. These values are tuned for a desktop pc with an AMD Ryzen 9 5900X. However as the values on the Intel Counter page were measured on an Intel 10th gen CPU and show similar IPC values these parameters might be less coupled to a specific system.

Our evaluation shows good results for this heuristic with better performance than schedutil for every workload. However the performance seems to be a bit worse compared to the L2-stalls/cycle based heuristic.

Power/Runtime evaluation:
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-erik-amd-ipc/evaluation.png)

Memutil log:
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-erik-amd-ipc/log-core0.png)

# Other evaluated heuristics
## Memory Stalls/Cycle bounds
We identified Memory Stalls/Cycle (cycle_activity.stalls_mem_any/cpu_clk_unhalted.thread on Intel) as a great metric to determine the level of memory-boundness a process is experiencing.
Because there can never be more stalls than cycles, this metric is normalized for any processor between 0 and 1. It also directly measures the wasted cycles that occured during any given operation, which we aim to minimize.

The first heuristic we tested with this metric is to try to target to keep it between two values. Initial testing suggested a value between 25% and 75%.
If the stalls/cycle exceed 75%, the CPU frequency ought to be reduced and if it is lower than 25%, the CPU frequency should be increased.
If the CPU is clocked slower, the stalls/cycle should decrease, as the memory system has more time to catch up. And if the CPU frequency is increased, the stalls should increase, as the CPU is going to request more memory faster.

This initial implementation showed promise on the Multi-Grid (mg) NAS Benchmark, where it chose a low frequency for the main task of the benchmark but still performed with a runtime close to the fastest frequency tested, whilst consuming even less energy than usual for its average frequency.
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-bounds-no-de-leon-laptop-nas-benchmarks/evaluation.png)

![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-bounds-no-de-leon-laptop-nas-benchmarks/log.png)
This second graph shows both the chosen frequency (between 0.8 and 2.6GHz), as well as the stalls/cycle (between 0-1 at the bottom) for each of the 8 cores.

We even see a slight benefit with memutil in the cg benchmark, however, as the stalls/cycle in this benchmark are only slightly above 75%, memutil doesn't reduce the CPU speed far enough.
This problem will likely remain for any heuristic that is based on such clear boundaries, as it is unclear where to set that boundary exactly.


## Stalls/Cycle + Instructions/Cycle
We found that memory-bound workloads tend to have low-IPC&high-Stalls/Cycle on high frequency. A heuristic might try to reduce frequency until IPC is high enough, but only if stalls/cycle is high (i.e. workload is memory-bound).

A heuristic based on this approach didn't seem to produce the desired results though, as certain tasks exhibited low-ipc&high memory stalls, whilst still not being off-core bound (i.e. the setup phase in mg.C), which caused memutil to chose frequencies too low for the workloads characteristics.

![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-ipc-spc-leon-laptop-nas/evaluation.png)

# Further ideas
## Normalized stalls/cycle
idea: Estimate what the stalls/cycle *would* have been, if we were to run at full frequency, then choose a frequency based on that (probably linearly). -> stalled time/time unit

Result: Possibly hard to normalize, workloads behave differently when changing frequency.
Integer Sort (is) for example shows no difference in stalls/cycle when changing frequency.
This might be explained by the memory unit being stalled, but there being enough other instructions to execute, so that the execution in total is not stalled.

The results are visualized in the following graph, where the requested frequency, the Instructions/Cycle & Stalls/Cycle are plotted.

There you can see how the different workloads behave differently, making it hard to normalize the stalls/cycle.

![[https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/normalize-stalls/evaluation/memutil-step-frequencies/log.png]]

**Update:** We conducted these measurements with the general memory stall performance counters. As discussed above, as well as on the [Intel Counters](Counters-Intel) page, it is better to divide the memory stalls into on-core (L1&L2) and off-core (L3&RAM) memory stalls.
So this approach of normalizing the stalls, based on the current frequency might be a feasible, but needs to be updated to use the L2 stalls instead of total memory stalls.

## Experimenting governor - reduce frequency and observe Instructions/Cycle
For a memory-bound workload, IPC should increase when reducing frequency.
Heuristic based on this=> reduce frequency, observe whether IPC rises, if so, reduce Frequency further, otherwise revert to higher frequency.

Advantage: Only needs 2, maybe even 0 perf counters (when using Instructions/time), instructions & cycles might not even be a perf counter, but a built-in fixed counter, so we wouldn't need perf counters at all.

Might be problematic for workloads that switch phases frequently, or run sporadically, as it would take longer to make decisions. Noise could also be problematic, as it might indicate wrong values.

## Combination of an immediate heuristic & the experimenting governor
Try to combine a heuristic that immediately chooses a frequency with the experimenting governor, so that we could detect&undo unfavorable decisions made by the immediate heuristic.

## mem_stalls/stalls_total
This might be a good way to figure out how truly memory-bound a workload is.
i.e. was the workload actually stalled due to memory, or something else.

Note: stalls_total could likely be measured with uops_retired.stalls_cycles or uops_executed.stall_cycles
