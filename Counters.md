# Measured Counters:

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


# Just Core 0

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

# All Cores - Less Colors

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