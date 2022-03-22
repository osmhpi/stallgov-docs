# Porting memutil

## Evaluating the ideal frequency.
First it is best to run the Evaluations to get a sense of which workloads perform best at which frequencies on your machine.

## Modifying and Running Memutil
Memutil with the l2stalls heuristic is currently tuned for a laptop based on the i7-6700HQ mobile processor.

To improve performance of memutil on your machine, first make sure you can [compile](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/kernel-module/README.md) memutil for your kernel and architecture (Intel and AMD only currently).

Then, edit `memutil_main.c`.
First, make sure the correct Heuristic is used: `HEURISTIC_OFFCORE_STALLS` is recommended for Intel CPUs, `HEURISTIC_IPC` for AMD.
The `#define HEURISTIC` must be set to the appropriate value.

### HEURISTIC_IPC
`memutil_main.c` contains two values: `max_freq_ipc` and `min_freq_ipc`.
These two values probably need to be adjusted for your machine.
`min_freq_ipc` is the IPC (in percent) for which memutil will choose the minimum frequency.
When `max_freq_ipc` is reached, memutil will choose the maximum frequency.
In-between values are interpolated linearly.

These values are exposed as kernel module parameters and can be tweaked when inserting the kernel module.
Therefore it is best to repeatedly insert memutil with different values, then run both an on-core and an off-core bound workload and check whether memutil chooses an appropriate frequency.

If memutil chooses a wrong frequency for the off-core (i.e. memory-bound) workload, adjust the `min_freq_ipc`.
Increasing the value should reduce the frequency, lowering it should increase the frequency.
The same goes for on-core (i.e. CPU-bound) workloads and the `max_freq_ipc` value.

### HEURISTIC_OFFCORE_STALLS

`memutil_main.c` contains two values: `max_freq_stalls_per_cycle` and `min_freq_stalls_per_cycle`.

These two value probably need to be adjusted for your specific machine.
`min_freq_stalls_per_cycle` is the Minimum l2stalls per cycle (In Percent 0-100) for which memutil will choose the minimum available frequency.
When `max_freq_stalls_per_cycle` is reached, memutil will choose the maximum available frequency.
In-between values are interpolated linearly.

These values are exposed as kernel module parameters and can be tweaked when inserting the kernel module.
Therefore it is best to repeatedly insert memutil with different values, then run both an on-core and an off-core bound workload and check whether memutil chooses an appropriate frequency.

If memutil chooses a wrong frequency for the off-core (i.e. memory-bound) workload, adjust the `min_freq_stalls_per_cycle`.
Increasing the value should reduce the frequency, lowering it should increase the frequency.
The same goes for on-core (i.e. CPU-bound) workloads and the `max_freq_stalls_per_cycle` value.
