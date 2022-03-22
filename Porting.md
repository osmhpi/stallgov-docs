# Porting memutil

## Evaluating the ideal frequency.
First it is best to run the Evaluations to get a sense of which workloads perform best at which frequencies on your machine.

## Modifying and Running Memutil
Memutil with the l2stalls heuristic is currently tuned for a laptop based on the i7-6700HQ mobile processor.

To improve performance of memutil on your machine, first make sure you can [compile](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/kernel-module/README.md) memutil for your kernel and architecture (Intel and AMD only currently).

Then to choose the [heuristic](Memutil-heuristics).

## Using L2-cache stalls

This is the default heuristic used and performs best. However the needed performance counters are not always available.
As default the counter `cycle_activity.stalls_l2_miss` is used. If this counter is not available `perf list` can be used to get a list of available counters and search for a replacement.
The alternative counter name can be passed to memutil as parameter as in `insmod memutil.ko event_name3=<replacement counter name>`. Note that it has to be `event_name3` as `event_name1` and `event_name2` are used for other counters.

## Using IPC

This heuristic peforms a bit worse, however it is usable on each computer as it does not use any platform specific counters.
To use this heuristic the line `#define HEURISTIC HEURISTIC_OFFCORE_STALLS` in memutil_main.c has to be changed to `#define HEURISTIC HEURISTIC_IPC`.

## Tuning the heuristic

This section assumes that you read the [heuristics page](Memutil-heuristics), especially the implementation details ([here](Memutil-heuristics#heuristic-implementation-details))

Both heuristics use a linear interpolation for which the parameters $`\beta_{max}`$ and $`\beta_{min}`$ can and probably should be tuned by passing their value as module parameter.

### HEURISTIC_IPC
Here $`\beta_{max}`$ is called `max_ipc` and if the IPC reaches this value the maximum frequency is choosen. $`\beta_{min}`$ is called `min_ipc`. If that IPC is reached, the minimum frequency will be choosen. Note that these values are given in percent (i.e. $`\beta_{min} = \frac{\text{min_ipc}}{100}`$) and should be between 0 and 100. Also `min_ipc` should be smaller than `max_ipc`.

Two pass these values to memutil, insert the module as follows: `insmod memutil.ko max_ipc=<some_value> min_ipc=<some_value>`.

It is best to repeatedly insert memutil with different values, then run both an on-core and an off-core bound workload and check whether memutil chooses an appropriate frequency.

If memutil chooses a wrong frequency for the off-core (i.e. memory-bound) workload, adjust the `min_ipc`.
Increasing the value should reduce the frequency, lowering it should increase the frequency.
The same goes for on-core (i.e. CPU-bound) workloads and the `max_ipc` value.

### HEURISTIC_OFFCORE_STALLS

Here $`\beta_{max}`$ is called `max_stalls_per_cycle` and if the L2-stalls/cycle reaches this value the **minimum** frequency is choosen. $`\beta_{min}`$ is called `min_stalls_per_cycle`. If that value is reached, the **maximum** frequency will be choosen. Note that these values are given in percent (i.e. $`\beta_{min} = \frac{\text{min_stalls_per_cycle}}{100}`$) and should be between 0 and 100. Also `min_stalls_per_cycle` should be smaller than `max_stalls_per_cycle`.

Two pass these values to memutil, insert the module as follows: `insmod memutil.ko max_stalls_per_cycle=<some_value> min_stalls_per_cycle=<some_value>`.

It is best to repeatedly insert memutil with different values, then run both an on-core and an off-core bound workload and check whether memutil chooses an appropriate frequency.

If memutil chooses a wrong frequency for the off-core (i.e. memory-bound) workload, adjust the `max_stalls_per_cycle`.
Decreasing the value should reduce the frequency, increasing it should increase the frequency.
The same goes for on-core (i.e. CPU-bound) workloads and the `min_stalls_per_cycle` value: Increasing it will increase the frequency while decreasing it will reduce the frequency.
