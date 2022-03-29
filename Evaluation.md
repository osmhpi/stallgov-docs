
For the evaluation of the memutil governor, we mostly focus on two metrics: "execution time" and "energy consumption".
We currently measure the energy consumption using the RAPL counters directly available within modern x86 CPUs.
These counters are very precise but only measure CPU energy consumption, not total system energy consumption.
That needs to be taken into consideration when discussing energy and execution time trade-offs.

A part of evaluating a DVFS CpuFreq governor must always be finding the ideal clock frequency for a given workload.
For this reason we don't just compare different governors, but also run any given workload at a range of frequencies.

A resulting diagram from these measurements typically looks like this:

![A typical Frequency-Runtime/Energy diagram](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

The average frequency of any given workload run (as measured by perf) is used as the X-axis value.
The Y-axis then contains two plots.
One for execution time and one for package energy consumed.

For the example above, multiple of these plots were combined into a single image to give an overview of a single evaluation run, including all workloads.

Some points to note about these graphs:
####  Check the range of the X-Axis
We create the measurements by running the "performance" governor and limiting its maximum frequency.
As explained in the [Future work page](Future Work), the Turbo Boost behavior of modern CPUs is out of the control of the CpuFreq governor.
Therefore the CPU might have boosted further when measuring certain workloads.
The range of the X-Axis can therefore be different between workloads which needs to be taken into account when evaluating governor behavior.

#### Fixed vs. dynamic frequency
The "frequency" and "execution time" curves are created by requesting a fixed frequency at which the workload is to be run.
As CpuFreq Governors like memutil or schedutil choose frequencies dynamically as needed they behave differently.
Therefore their execution time or energy consumption may be outside the curve of fixed frequencies.
As they don't run at just a single frequency, the X-Axis value of a governor measurement are using the average frequency that the workload actually ran at.

All frequency measurements may also differ from the frequency requested by the governor as we measure the actual frequency during program execution.
As the CPU driver has final control over the exact DVFS behavior of the cores, it might have overriden the frequency.
We measure and plot the actual frequency, not the requested frequency.

## Our Evaluation Framework


## Memutil Evaluation & Comparison

## Future Work

Measure total system power, not just RAPL counter.
Our expectation is that this will push the optimal frequency a bit higher, as improvements in execution time will save total system energy due to shorter power consumption by memory, GPU, mainboard, etc.
