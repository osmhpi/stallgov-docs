For the development of Memutil, we are testing & evaluating different heuristics, to achieve the optimal performance/Watt. The different heuristics we came up with are explained here.

## General problem
Figure out how memory-bound a workload is.
Complication: high memory-stalls != memory-bound -> memory unit might be stalled, but enough other instructions to execute.

## Stalls/Cycle bounds
We identified Stalls/Cycle (cycle_activity.stalls_mem_any/cpu_clk_unhalted.thread on Intel) as a great metric to determine the level of memory-boundness a process is experiencing.
Because there can never be more stalls than cycles, this metric is normalized for any processor between 0 and 1. It also directly measures the wasted cycles that occured during any given operation, which we aim to minimize.

The first heuristic we tested with this metric is to try to target to keep it between two values. Initial testing suggested a value between 25% and 75%.
If the stalls/cycle exceed 75%, the CPU frequency ought to be reduced and if it is lower than 25%, the CPU frequency should be increased.
If the CPU is clocked slower, the stalls/cycle should decrease, as the memory system has more time to catch up. And if the CPU frequency is increased, the stalls should increase, as the CPU is going to request more memory faster.

This initial implementation showed promise on the Multi-Grid (mg) NAS Benchmark, where it chose a low frequency for the main task of the benchmark but still performed with a runtime close to the fastest frequency tested, whilst consuming even less energy than usual for its average frequency.
![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/memutil-bounds-no-de-leon-laptop-nas-benchmarks/evaluation.png)

![](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/memutil-bounds-no-de-leon-laptop-nas-benchmarks/log.png)
This second graph shows both the chosen frequency (between 0.8 and 2.6GHz), as well as the stalls/cycle (between 0-1 at the bottom) for each of the 8 cores.

We even see a slight benefit with memutil in the cg benchmark, however, as the stalls/cycle in this benchmark are only slightly above 75%, memutil doesn't reduce the CPU speed far enough.
This problem will likely remain for any heuristic that is based on such clear boundaries, as it is unclear where to set that boundary exactly.



## Further ideas
### Normalized stalls/cycle
idea: Estimate what the stalls/cycle *would* have been, if we were to run at full frequency, then choose a frequency based on that (probably linearly). -> stalled time/time unit

Result: Possibly hard to normalize, workloads behave differently when changing frequency.
Integer Sort (is) for example shows no difference in stalls/cycle when changing frequency.
This might be explained by the memory unit being stalled, but there being enough other instructions to execute, so that the execution in total is not stalled.

The results are visualized in the following graph, where the requested frequency, the Instructions/Cycle & Stalls/Cycle are plotted.

There you can see how the different workloads behave differently, making it hard to normalize the stalls/cycle.

![[https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/normalize-stalls/evaluation/memutil-step-frequencies/log.png]]

### Stalls/Cycle + Instructions/Cycle
We found that memory-bound workloads tend to have low-IPC&high-Stalls/Cycle on high frequency. A heuristic might try to reduce frequency until IPC is high enough, but only if stalls/cycle is high (i.e. workload is memory-bound).

### Experimenting governor - reduce frequency and observe Instructions/Cycle
For a memory-bound workload, IPC should increase when reducing frequency.
Heuristic based on this=> reduce frequency, observe whether IPC rises, if so, reduce Frequency further, otherwise revert to higher frequency.

Advantage: Only needs 2, maybe even 0 perf counters (when using Instructions/time), instructions & cycles might not even be a perf counter, but a built-in fixed counter, so we wouldn't need perf counters at all.

Might be problematic for workloads that switch phases frequently, or run sporadically, as it would take longer to make decisions. Noise could also be problematic, as it might indicate the also be replacedwrong values.

### Combination of an immediate heuristic & the expirmenting governor
Try to combine a heuristic that immediately chooses a frequency with the experimenting governor, so that we could detect&undo unfavorable decisions made by the immediate heuristic.

### mem_stalls/stalls_total
This might be a good way to figure out how truly memory-bound a workload is.
i.e. was the workload actually stalled due to memory, or something else.

Note: stalls_total could likely be measured with uops_retired.stalls_cycles or uops_executed.stall_cycles