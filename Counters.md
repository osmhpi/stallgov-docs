For the evaluation of the measurements, see the "Evaluation" section at the [bottom of this page](#evaluation)

# Intel

## Measured Counters:

| Title | Counter 1 | Counter 2 | Counter 3 | plot_op |
| ----- | --------- | ----------| --------- | ------- |
| L1_Pend_Stall_Cycles | cycle_activity.cycles_l1d_miss | cycle_activity.stalls_l1d_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| L2_Pend_Stall_Cycles | cycle_activity.cycles_l2_miss | cycle_activity.stalls_l2_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| L3_Pend_Stall_Cycles | cycle_activity.cycles_l3_miss | cycle_activity.stalls_l3_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| L1_Hitrate | mem_load_retired.l1_hit | mem_load_retired.l1_miss | cpu_clk_unhalted.thread | div_1_(1+2) |
| L2_Hitrate | mem_load_retired.l2_hit | mem_load_retired.l2_miss | cpu_clk_unhalted.thread | div_1_(1+2) |
| L3_Hitrate | mem_load_retired.l3_hit | mem_load_retired.l3_miss | cpu_clk_unhalted.thread | div_1_(1+2) |
| U_Execute-Stall_vs_Mem-Stall | uops_executed.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| U_Retired-Stall_vs_Mem-Stall | uops_retired.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| U_Issued-Stall_vs_Mem-Stall | uops_issued.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| U_Resource-Stall_vs_Mem-Stall | resource_stalls.any | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| Stall_Total_vs_Mem-Stall | cycle_activity.stalls_total | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |
| Divider cycles vs IPC | arith.divider_active | inst_retired.any | cpu_clk_unhalted.thread | div_1_3+div_2_3 |


```python
counters = [{'counter': ['cycle_activity.cycles_l1d_miss', 'cycle_activity.stalls_l1d_miss', 'cpu_clk_unhalted.thread'], 'title': 'L1_Pend_Stall_Cycles', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Pending', 'Stalls']},
            {'counter': ['cycle_activity.cycles_l2_miss', 'cycle_activity.stalls_l2_miss', 'cpu_clk_unhalted.thread'], 'title': 'L2_Pend_Stall_Cycles', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Pending', 'Stalls']},
            {'counter': ['cycle_activity.cycles_l3_miss', 'cycle_activity.stalls_l3_miss', 'cpu_clk_unhalted.thread'], 'title': 'L3_Pend_Stall_Cycles', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Pending', 'Stalls']},
            {'counter': ['mem_load_retired.l1_hit', 'mem_load_retired.l1_miss', 'cpu_clk_unhalted.thread'], 'title': 'L1_Hitrate', 'plot_op': 'div_1_(1+2)', 'plot_names': ['Hitrate']},
            {'counter': ['mem_load_retired.l2_hit', 'mem_load_retired.l2_miss', 'cpu_clk_unhalted.thread'], 'title': 'L2_Hitrate', 'plot_op': 'div_1_(1+2)', 'plot_names': ['Hitrate']},
            {'counter': ['mem_load_retired.l3_hit', 'mem_load_retired.l3_miss', 'cpu_clk_unhalted.thread'], 'title': 'L3_Hitrate', 'plot_op': 'div_1_(1+2)', 'plot_names': ['Hitrate']},
            {'counter': ['uops_executed.stall_cycles', 'cycle_activity.stalls_mem_any', 'cpu_clk_unhalted.thread'], 'title': 'U_Execute-Stall_vs_Mem-Stall', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Execute Stalls', 'Mem Stalls']},
            {'counter': ['uops_retired.stall_cycles', 'cycle_activity.stalls_mem_any', 'cpu_clk_unhalted.thread'], 'title': 'U_Retired-Stall_vs_Mem-Stall', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Retired Stalls', 'Mem Stalls']},
            {'counter': ['uops_issued.stall_cycles', 'cycle_activity.stalls_mem_any', 'cpu_clk_unhalted.thread'], 'title': 'U_Issued-Stall_vs_Mem-Stall', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Issued Stalls', 'Mem Stalls']},
            {'counter': ['resource_stalls.any', 'cycle_activity.stalls_mem_any', 'cpu_clk_unhalted.thread'], 'title': 'U_Resource-Stall_vs_Mem-Stall', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Resource Stalls', 'Mem Stalls']},
            {'counter': ['cycle_activity.stalls_total', 'cycle_activity.stalls_mem_any', 'cpu_clk_unhalted.thread'], 'title': 'Stall_Total_vs_Mem-Stall', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Stalls Total', 'Mem Stalls']},
            {'counter': ['arith.divider_active', 'inst_retired.any', 'cpu_clk_unhalted.thread'], 'title': 'Divider cycles vs IPC', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Divider Cycles', 'IPC']}]
```

## Just Core 0

![complete](uploads/5c32071b5032e5bf3c2d00280b47bb33/complete.png)

![complete](uploads/d4699ce9daf6a248351e1fb87fea7694/complete.png)

![complete](uploads/97c1de145de82d75ddd1dd5152943f10/complete.png)

![complete](uploads/bec6f16f32f5b9d066d0a12dace607f7/complete.png)

![complete](uploads/4d9b1432a9122a70c1eeb617fcbb8a0d/complete.png)

![complete](uploads/4cd2828a3af94c9b8711be61edfe174d/complete.png)

![complete](uploads/5804c302f67b4d96b24d856e3ff91c96/complete.png)

![complete](uploads/133733201888e7ca39b03961a8a12437/complete.png)

![complete](uploads/648cd05585f69c814c42a827656c378c/complete.png)

![complete](uploads/160c0cbb42cffea2a238fb11d837c469/complete.png)

![complete](uploads/38a34e6c40e02ddd5e61b60df446ec53/complete.png)

![complete](uploads/3476805123d6de9a62acccc804d9d469/complete.png)

# All Cores

![complete](uploads/9d1f90ba70adf02e53f4bcedca7fdbf5/complete.png)

![complete](uploads/40c8dac9e59fa66be183f1837f152c15/complete.png)

![complete](uploads/28c716034adf20f6d5b61399c353986e/complete.png)

![complete](uploads/0e7ff7ac3826f904e841aa4a1571cd2c/complete.png)

![complete](uploads/4bfa3bf1fcbf6a65f202082ff89c68aa/complete.png)

![complete](uploads/a556fb0d3af331d4befa200449cff24b/complete.png)

![complete](uploads/71ae579c9dcd0d3fd1eef49a63e0a0d3/complete.png)

![complete](uploads/986b6d2406fdd5b18529c5e476c3ce26/complete.png)

![complete](uploads/f8573cf0a41f684bb61301c0dddead8e/complete.png)

![complete](uploads/dfaad079146388b0a1c12998dfea87f5/complete.png)

![complete](uploads/1c31c1a0fc03937c3b68226f4ef36e05/complete.png)

![complete](uploads/47a4f5696a085e13fb73c8e2603efd92/complete.png)

## All Cores - Less Colors

![complete](uploads/2187fcf3682ac7d973a340f184325c6c/complete.png)

![complete](uploads/7486009133d9e146c54dc6cabb8a866f/complete.png)

![complete](uploads/492351ac5bc27a3757a3bc83fde00445/complete.png)

![complete](uploads/8471b4dfdcb31a75aae52dc4002d1700/complete.png)

![complete](uploads/df9b93968fbc5be9a684f74591604028/complete.png)

![complete](uploads/857085ed9449de35cf0bfd03c51f8b05/complete.png)

![complete](uploads/cc1efc0de21e495ccc3a961a89dec011/complete.png)

![complete](uploads/e6d7b7ae6111656335e189adb102f1a3/complete.png)

![complete](uploads/a1f077976ba22c6712b50ff67e860518/complete.png)

![complete](uploads/e0de912212f94527160595acd48e619a/complete.png)

![complete](uploads/846da2bff070c999c99fe54b46d62448/complete.png)

![complete](uploads/0cf2bc9ab61727896de4b07a5c16d8c3/complete.png)

# AMD

## Measured Counters:

```python
counters = [{'counter': ['l2_fill_pending.l2_fill_busy', 'l2_latency.l2_cycles_waiting_on_fills', 'cycles'], 'title': 'L2_Fill_Pending', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Pending', 'Latency']},
            {'counter': ['ls_dispatch.ld_dispatch', 'ls_dispatch.ld_st_dispatch', 'instructions'], 'title': 'Load Store Inst', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Load', 'Load-Store']},
            {'counter': ['l2_cache_req_stat.ic_dc_miss_in_l2', 'l2_cache_req_stat.ic_dc_hit_in_l2', 'cycles'], 'title': 'L2_Hitrate', 'plot_op': 'div_1_(1+2)', 'plot_names': ['Hitrate']},
            {'counter': ['stalled-cycles-backend', 'stalled-cycles-frontend', 'cycles'], 'title': 'Backend-Frontend-Stalls', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Backend', 'Frontend']},
            {'counter': ['instructions', 'cycles', 'cycles'], 'title': 'IPC', 'plot_op': 'div_1_2', 'plot_names': ['IPC']},
            {'counter': ['l3_cache_accesses', 'l3_misses', 'cycles'], 'title': 'L3 per Cycle', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Accesses per Cycle', 'Misses per Cycle']},
            {'counter': ['l2_cache_req_stat.ic_dc_miss_in_l2', 'l2_cache_req_stat.ic_dc_hit_in_l2', 'cycles'], 'title': 'L2 per Cycle', 'plot_op': 'div_1_3+div_2_3', 'plot_names': ['Misses per Cycle', 'Hits per Cycle']}]



```

## All Cores

![complete](uploads/77851d05bf9d55eaba4bc525fd4ef8f2/complete.png)

![complete](uploads/d3f7c3e4be4ff8991e46402b308c0040/complete.png)

![complete](uploads/f9c2bf3c7457603fad5b3e415b1e5901/complete.png)

![complete](uploads/8ab099215027feed77a0c4eef7f94e84/complete.png)

![complete](uploads/ba42dbb727ad9ed7ca63d27bbdcf65e8/complete.png)

![complete](uploads/a52da1dc1eebbb7ccbe0853c5aa4b532/complete.png)

![complete](uploads/f40924bfdad4b2129a192e51eb291317/complete.png)

## Only L2_Fill_Pending Pending plot

![complete](uploads/f4b575f219e8a89cebde7cfcf42a5c37/complete.png)

# Evaluation
The point of these measurements is to identify hardware performance counters that reliably indicate which tasks benefit from reducing the cores clock frequency.
This is beneficial if a task is bound by the memory subsystem (L3 cache & main memory), which doesn't depend on the clock frequency of a single core. Therefore reducing the clock frequency will not slow down the task and only save power.

We have thus far been able to categorize the different workloads depending on how much they benefit from a reduced clock frequency:
1. mg.C - mostly heavily memory-bound, except during a short startup-phase
2. cg.B - mostly memory-bound, but less so than mg.C
3. ft.B - in-between, benefiting a lot in runtime performance with increased clock frequency - up to a point.
4. ep.B - cpu-bound workload - highest frequency here is best
5. is.C - memory-bound workload, however this workload does *not* benefit much from a reduced frequency!

Most of the counters we measured could correctly identify mg.C, cg.B, as well as is.C as memory-bound, as they are all measuring a significant number of stall cycles due to memory overhead.
However, the evaluation of the is.C (integer-sort) workload (as seen on the [Heuristics](Memutil heuristics) page) indicates that is.C is memory-bound, yet still benefits from an increased clock core frequency.

Our current working theory is, that this is due to this workload accessing the L1, as well as L2 cache a lot. These caches always belong to a certain core and are clocked with the same frequency, therefore benefiting from the increase in clock frequency.
However, most of the counters relating to "memory stalls" or other "resource stalls" still treat stalls on these caches as "memory stalls".
The outlier here though is the number of memory stalls caused by l2-misses, which only identifies mg.C and cg.B as stalling a lot, whilst is.C almost never stalls on l2 misses.

Therefore our heuristic should focus on the counters related to l2 misses, as they directly correlate with the number of l3 cache or main memory accesses that occur. These frequency domains are not influenced by individual core frequencies.
To better separate the workloads, we will therefore refer to them as on-core bound (meaning bound by anything within the cores frequency domain, including L1&L2 caches), as well as off-core bound, which mainly refers to L3 cache & main memory, but could also be expanded to include other resources in the future, like network or hard drive accesses.
We might still refer to memory-bound or cpu-bound workloads in our documentation, which should be read as off-core-bound and on-core-bound respectively in most cases.

To read up on the heuristics resulting from these measurements, see the [Memutil Heuristics](Memutil heuristics) page.