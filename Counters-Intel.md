For the evaluation of the measurements, see the "Evaluation" section at the [bottom of this page](#evaluation)

# Measurement setup:

We used our memutil governor to measure the values of the selected perf events. For this we changed memutil to always set the max frequency in the update frequency step and just log the counters. Additionally we adjusted our scripts to just do memutil measurements. Lastly additional scripts where used to simplify measurement and plotting with one command.

The modified `memutil_main.c` as well as the modified scripts are available in the `evaluation/counter-measurements` folder. 

# Measured Counters:

| Title | Counter 1 | Counter 2 | Counter 3 | plot_op | plot names |
| ----- | --------- | ----------| --------- | ------- | ---------- |
| L1_Pend_Stall_Cycles | cycle_activity.cycles_l1d_miss | cycle_activity.stalls_l1d_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Pending, Stalls |
| L2_Pend_Stall_Cycles | cycle_activity.cycles_l2_miss | cycle_activity.stalls_l2_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Pending, Stalls |
| L3_Pend_Stall_Cycles | cycle_activity.cycles_l3_miss | cycle_activity.stalls_l3_miss | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Pending, Stalls |
| L1_Hitrate | mem_load_retired.l1_hit | mem_load_retired.l1_miss | cpu_clk_unhalted.thread | div_1_(1+2) | Hitrate |
| L2_Hitrate | mem_load_retired.l2_hit | mem_load_retired.l2_miss | cpu_clk_unhalted.thread | div_1_(1+2) | Hitrate |
| L3_Hitrate | mem_load_retired.l3_hit | mem_load_retired.l3_miss | cpu_clk_unhalted.thread | div_1_(1+2) | Hitrate |
| U_Execute-Stall_vs_Mem-Stall | uops_executed.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Execute Stalls, Mem Stalls |
| U_Retired-Stall_vs_Mem-Stall | uops_retired.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Retired Stalls, Mem Stalls |
| U_Issued-Stall_vs_Mem-Stall | uops_issued.stall_cycles | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Issued Stalls, Mem Stalls |
| U_Resource-Stall_vs_Mem-Stall | resource_stalls.any | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Resource Stalls, Mem Stalls |
| Stall_Total_vs_Mem-Stall | cycle_activity.stalls_total | cycle_activity.stalls_mem_any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Stalls Total, Mem Stalls |
| Divider cycles vs IPC | arith.divider_active | inst_retired.any | cpu_clk_unhalted.thread | div_1_3+div_2_3 | Divider Cycles, IPC |


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

The table shows which counters we measured and how we plotted them. The title is the title of the plot. The next 3 columns identifies which counters we measured. The plot ops mean the following:
 - `div_1_3+div_2_3` One plot of $`counter1 / counter3`$ and one of $`counter2 / counter3`$
 - `div_1_(1+2)` One plot for $`\frac{counter1}{counter2 + counter3}`$

The plot names are given in order, i.e. the first name is used for the first plot, the second name is used for the second plot, etc.

# Just Core 0

![complete](uploads/97c1de145de82d75ddd1dd5152943f10/complete.png)

![complete](uploads/4d9b1432a9122a70c1eeb617fcbb8a0d/complete.png)

![complete](uploads/5804c302f67b4d96b24d856e3ff91c96/complete.png)

![complete](uploads/d4699ce9daf6a248351e1fb87fea7694/complete.png)

![complete](uploads/bec6f16f32f5b9d066d0a12dace607f7/complete.png)

![complete](uploads/4cd2828a3af94c9b8711be61edfe174d/complete.png)

![complete](uploads/648cd05585f69c814c42a827656c378c/complete.png)

![complete](uploads/3476805123d6de9a62acccc804d9d469/complete.png)

![complete](uploads/160c0cbb42cffea2a238fb11d837c469/complete.png)

![complete](uploads/38a34e6c40e02ddd5e61b60df446ec53/complete.png)

![complete](uploads/133733201888e7ca39b03961a8a12437/complete.png)

![complete](uploads/5c32071b5032e5bf3c2d00280b47bb33/complete.png)

# All Cores

![complete](uploads/846da2bff070c999c99fe54b46d62448/complete.png)

![complete](uploads/492351ac5bc27a3757a3bc83fde00445/complete.png)

![complete](uploads/8471b4dfdcb31a75aae52dc4002d1700/complete.png)

![complete](uploads/e0de912212f94527160595acd48e619a/complete.png)

![complete](uploads/7486009133d9e146c54dc6cabb8a866f/complete.png)

![complete](uploads/0cf2bc9ab61727896de4b07a5c16d8c3/complete.png)

![complete](uploads/857085ed9449de35cf0bfd03c51f8b05/complete.png)

![complete](uploads/a1f077976ba22c6712b50ff67e860518/complete.png)

![complete](uploads/cc1efc0de21e495ccc3a961a89dec011/complete.png)

![complete](uploads/e6d7b7ae6111656335e189adb102f1a3/complete.png)

![complete](uploads/df9b93968fbc5be9a684f74591604028/complete.png)

![complete](uploads/2187fcf3682ac7d973a340f184325c6c/complete.png)

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