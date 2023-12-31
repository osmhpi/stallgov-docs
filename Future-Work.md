# Future Work
## Idle Behavior
As we have demonstrated, memutil can save a significant portion of energy compared to the schedutil governor.
However, memutil currently only tracks how much the processor is stalled by memory access, not how much overall load it is experiencing.

Therefore, if the processor has little load, our theory is that this will result in few memory stalls, which memutil interprets as running an on-core-bound process, which leads to increased frequency, even though the small load would have still executed quickly enough on a lower clock frequency.
Generally, idle behavior needs to be examined more closely.
Especially studying the combined effects of sleep states together with DVFS behavior is not in the scope of this project.

### Combining memutil&schedutil
Schedutil currently deals well with system idle, as it makes decisions based on scheduler information.
A viable approach could therefore be to combine the benefits of memutil & schedutil, i.e. by running both governor algorithms asynchronously and choosing the minimum power state of the two algorithm outputs.
This way, the reduced clock frequency chosen by schedutil will be used in idle, and memutil may reduce the frequency for workloads that fully load the CPU, but don't benefit from increased clock frequencies.

It still needs to be evaluated if this approach is viable, or whether it will produce too much overhead by combining two governor algorithms.

## Controlling Turbo boost behavior
During our research we have noticed that modern laptop CPUs start loosing efficiency, even in on-core-bound workloads, once a certain performance-state is reached.
This is in contrast to previous research, which suggested that an on-core-bound workload is most efficient at the maximum clock frequency.

Unfortunately, on modern Intel CPUs the Intel Turbo-Boost-Technology is responsible for controlling the upper power states.
The CpuFreq Governor may only choose a frequency up to the CPUs "base frequency". If this frequency is selected, Intel Turbo-Boost may choose any frequency within the turbo range.
The actual frequency is then typically limited by the capabilities of the CPUs cooler.

This means that memutil currently cannot control the frequency within this turbo range.
For modern CPUs the turbo range can include a large range of frequencies, e.g. the i7-1165G7 has a turbo range from 2.8 to 4.7 GHz, so almost a third of the entire frequency range, which cannot be controlled by memutil.

It could therefore be beneficial if the turbo range could be controlled by memutil to gain control of the higher frequency ranges.
I.e. to limit turbo boost from boosting too high, as that reduces the efficiency on all processors we tested.

As mentioned in the [Architecture page](Memutil Architecture), it is also possible that the CpuFreq P-state API controls the entire frequency range, including the turbo range.
Therefore adding support for this API might improve the situation significantly.

## Improved AMD support

As described in the [Memutil Heuristics](Memutil Heuristics) page, the current best heuristic we found is the l2stalls/cycle.
Unfortunately, the l2stalls counter is a specific to Intel CPUs.
Therefore we currently cannot replicate the same heuristic on our available AMD CPUs.

We are still working on evaluating the available counters on AMD - see the [appropriate Counters](Counters AMD) page.

## Automatic Calibration

Currently memutil still needs to be calibrated for each machine manually.
This process is tedious, error prone and imprecise.
Ideally, memutil would automatically calibrate itself for the system it is running on.
For this, a list of predefined workloads could be run to measure the l2stalls/cycle as well as IPC on the current machine.
The heuristic values could then be adapted as indicated by the measurements.
Even better would be to continually measure the effects of memutil on power consumption and improve the heuristic parameters on-the-fly.
