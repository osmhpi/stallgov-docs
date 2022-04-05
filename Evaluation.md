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
As CpuFreq governors like memutil or schedutil choose frequencies dynamically as needed, they behave differently.
Therefore their execution time or energy consumption may be outside the curve of fixed frequencies.
As they don't run at just a single frequency, the X-Axis value of a governor measurement are using the average frequency that the workload actually ran at.

All frequency measurements may also differ from the frequency requested by the governor as we measure the actual frequency during program execution.
As the CPU driver has final control over the exact DVFS behavior of the cores, it might have overriden the frequency.
We measure and plot the actual frequency, not the requested frequency.

## Our Evaluation Framework
The [`evaluation`](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/tree/master/evaluation) folder of our repository contains all the code necessary to measure a workloads energy, execution time and log at different frequencies and with different governors.

See the README files in the `evaluation` folder for an explanation & usage of the different scripts.

#### Dependencies
- GraphicsMagick
- Pandas (Python package)
- matplotlib (Python package)

## Memutil Evaluation & Comparison
Our goal for memutil is to reduce total energy consumed by a given workload.
The execution time of the workload is currently not our priority.
Our target platform is a modern laptop that runs on battery power.
Therefore saving battery power is more important than execution time.
Typical workloads, like compiling code or running tests can be done in the background, whilst the device is still in normal
when the device is on battery power the workloads don't need to complete as fast as possible.
From our perspective it is more important that the device can stay operational for a longer period of time.
For this reason, we measure the total energy consumed by a given workload in Joules.

To gain a broader picture of the workloads energy consumption characteristics, we run each workload at different frequencies and with the schedutil, as well as memutil governors.
To ensure we don't accidentally measure an outlier, we run the workloads 3 times for any given frequency/governor.
The `evaluate_benchmarks.sh` script performs these measurements for a given list of workloads.

Running the `plot_multiple.sh` script then visualizes the measurements in two graphs: `evaluation.png` and `log.png`.
The `evaluation.png` file contains the energy and execution time measurements, whilst `log.png` visualizes the memutil logs from all workload runs.

### NAS benchmarks

#### evaluation.png
![A typical Frequency-Runtime/Energy diagram](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

#### log.png
![A typical Frequency-Runtime/Energy diagram](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/log.png)

## Future Work
Measure total system power, not just RAPL counter.
Our expectation is that this will push the optimal frequency a bit higher, as improvements in execution time will save total system energy due to shorter power consumption by memory, GPU, mainboard, etc.
