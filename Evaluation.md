For the evaluation of the memutil governor, we mostly focus on two metrics: "execution time" and "energy consumption".
We currently measure the energy consumption using the RAPL counters directly available within modern x86 CPUs.
These counters are very precise but only measure CPU energy consumption, not total system energy consumption.
That needs to be taken into consideration when discussing energy and execution time trade-offs.

A part of evaluating a DVFS CpuFreq governor must always be finding the ideal clock frequency for a given workload.
For this reason we don't just compare different governors, but also run any given workload at a range of frequencies.

A resulting diagram from these measurements typically looks like this:

![A typical Frequency/Exection time-Energy diagram](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

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
Our target platform is a contemporary laptop that runs on battery power.
Therefore saving battery power is more important than execution time.
Typical workloads, like compiling code or running tests can be done in the background, whilst the device is still in normal use.
When the device is on battery power the workloads don't need to complete as fast as possible.
From our perspective it is more important that the device can stay operational for a longer period of time, not how fast the workloads run.
For this reason, we measure the total energy consumed by a given workload in Joules.

To gain a broader picture of the workloads energy consumption characteristics, we run each workload at different frequencies and with the schedutil, as well as memutil governors.
To ensure we don't accidentally measure an outlier, we run the workloads 3 times for any given frequency/governor.
The `evaluate_benchmarks.sh` script performs these measurements for a given list of workloads.

Running the `plot_multiple.sh` script then visualizes the measurements in two graphs: `evaluation.png` and `log.png`.
The `evaluation.png` file contains the energy and execution time measurements, whilst `log.png` visualizes the memutil logs from all workload runs.

### NAS benchmarks
We chose 5 workloads from the NAS benchmark suite for our evaluation:

1. cg: Conjugate Gradient
2. ep: Embarrassingly parallel
3. ft: Fourier transform
4. is: Integer sort
5. mg: Multi-Grid

These workloads can all run in parallel and can therefore simulate a full system load.

#### evaluation.png
![Evaluation graphic for the NAS benchmarks](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/evaluation.png)

#### log.png
![Memutil logs for the NAS benchmarks](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/log.png)

The measurements in these graphs are from the `evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/` folder.
They were conducted on a Gigabyte Aero 14 (GTX 1060) with an i7-6700HQ running Fedora Linux on Kernel 5.15.
Memutil used the L2Stalls heuristic.

For this particular machine, the `mg` and `cg` workloads are the only ones that were clearly off-core-bound as they show only small execution time benefit after a certain frequency is reached.
`ft` seems to be a mix of off-core, as well as on-core bound, as it still benefits from increased frequency in execution time but starts to consume more energy relatively early in the frequency range.

For the NAS benchmarks, memutil correctly reduced the frequency for `mg` and `cg`, and yielded reduced energy consumption compared to schedutil by ~20% and ~29% respectively.
The execution time was increased by &lt;1% for `mg` and by 26% for `cg`.
For `ft`, energy consumption was reduced by ~14%, whilst execution time increased by ~6%.

In all cases where memutil deviated from schedutils behavior it was able to save more energy than it increased execution time.
The execution time/energy curves for `cg` also lead us to believe that with further tuning, memutil could save a similar amount of energy whilst improving execution time penalty to &lt;10%.
These measurements confirm that to reduce energy consumption the current DVFS behavior of the schedutil CpuFreq Governor can be improved by taking memory behavior of the workload into account.

Also note how even on-core-bound workloads start to increase in energy usage once a certain threshold voltage is reached.
This is not entirely in line with previous research, which suggests that for on-core-bound workloads, increasing the frequency will result in lower energy consumption, as the CPU can return to idle sooner.
We also noticed that for the i7-6700HQ, this frequency seems to be close to the Turbo-Boost threshold voltage of 2.6GHz, at which point the CPU frequency leaves its base frequency.
Unfortunately the CpuFreq Governor can not control this frequency range, which is why we do not consider it fault of memutil, that it ran at sub-optimal frequencies there.
It remains future work to evaluate whether it is possible to control the turbo boost behavior.

### OpenBenchmarking tests
After tuning memutil for the NAS benchmark suite we used the OpenBenchmarking test set, instrumented by the Phoronix Test suite, to try and confirm the positive results achieved in the NAS benchmarks.

The code we used to run these benchmarks can be found in the [`evalution/openbenchmarking`](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/tree/master/evaluation/openbenchmarking) folder.

#### evaluation.png
![Evaluation graphic for the OpenBenchmarking tests](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-openbenchmarking/evaluation.png)

#### log.png
![Memutil log graphic for the OpenBenchmarking tests](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/raw/master/evaluation/results/memutil-l2stall-lerp-leon-laptop-openbenchmarking/log.png)

The measurements in these graphs are from the `evaluation/results/memutil-l2stall-lerp-leon-laptop-nas/` folder.
They were conducted on a Gigabyte Aero 14 (GTX 1060) with an i7-6700HQ running Fedora Linux on Kernel 5.15.
Memutil used the L2Stalls heuristic.

If you take a closer look at the x-axes of the `evaluation.png`, many of the measurements only have a very small frequency range.
This is likely caused by the benchmarks not utilizing the system fully, which causes idle time to skew our measurements.
As we currently don't know what the exact cause of this behavior is and do not know the ideal frequency for these workloads it is hard to draw conclusions from these measurements.

The measurements that showed a complete frequency range all seemed to be on-core bound - apart from sysbench - and therefore reducing the frequency does not grant any benefit.
memutil did correctly identify these workloads as on-core-bound, but therefore behaved similar to schedutil.

Note that sysbench seemed to be highly off-core-bound, but did not produce many l2stalls.
Therefore memutil chose a frequency that is too high.
Further research is needed into which performance counters could correctly identify sysbench as off-core-bound.

Generally, we still need to improve both our evaluation as well as memutils behavior for workloads that do not fully load our system.
We theoretically have the problem here that the idle process doesn't produce a lot of memory stalls, therefore memutil will choose the maximum frequency, which is of course not ideal.
See the [Future Work](Future Work) page for this as well.

## Conclusion
For workloads that fully load all cores of our system, memutil can already achieve significant energy savings of over 20%.
We currently cannot validate these findings on the OpenBenchmarking test suite, as the test suite did not fully load our system.
Further research is needed into memutils behavior at idle, especially the effects of DVFS in these cases, especially in combination with processor sleep states.
Both our evaluation, as well as memutil itself must therefore still be improved for workloads that spend a lot of time idle.

We still conclude that energy efficiency can be improved by taking memory access characteristics into account, given a workload which doesn't idle and is bound by memory that is not part of the cores frequency domain.
memutil also has potential for improvement with better calibration, as demonstrated by the "Conjugate Gradient" NAS benchmark.

## Future Work
- Evaluate the measurements done on AMD using the IPC heuristic.
- Improve memutil behavior & evaluation at idle.
- Measure total system power, not just RAPL counter.\
Our expectation is that this will push the optimal frequency a bit higher, as improvements in execution time will save total system energy due to shorter power consumption by memory, GPU, mainboard, etc.
