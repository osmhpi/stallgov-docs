# Evaluation

For the evaluation of the memutil governor, we mostly focus on two metrics: "execution time" and "energy consumption".
We currently measure the energy consumption using the RAPL directly available within modern x86 CPUs.
These counters are very precise but only measure CPU energy consumption, not total system energy consumption.
That needs to be take into consideration when discussing energy and execution time trade-offs.

As part of evaluating a DVFS CpuFreq governor must always be finding the ideal clock frequency for a given workload, we don't just compare different governors, but also run any given workload at a range of frequencies.

A resulting diagram from these measurements typically looks like this:

![A typical Frequency-Runtime/Energy diagram](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

The average frequency of any given workload run (as measured by perf) is used as the X-axis value.
The Y-axis then contains two plots.
One for execution time and one for package energy consumed.

For the example above, multiple of these plots were combined into a single image to give an overview of a single evaluation run, including all workloads.



## Future Work

Measure total system power, not just RAPL counter.
Our expectation is that this will push the optimal frequency a bit higher, as improvements in runtime will save total system energy due to shorter power consumption by memory, GPU, mainboard, etc.
