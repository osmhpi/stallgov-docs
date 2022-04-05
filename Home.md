This is the homepage of the project: "Energy-Aware computing: Cut compiler carbon footprint"

We are focusing on creating a new Linux [CpuFreq Governor](https://www.kernel.org/doc/html/latest/cpu-freq/index.html) that makes dynamic voltage scaling decisions based on hardware performance counters. This will allow it to react to the memory access patterns of the current CPU processes, so that it can reduce clock frequency, should the CPU be stalled by memory.

## Background & Related Work
See [this page](Background-&-Related-Work)

## memutil
Our CpuFreq Governor is called memutil.
It is based on hardware performance counters to determine memory-boundness of the currently active processes.
Therefore it is probably best if you familiarize yourself with the different hardware performance counters we measured.
The different Hardware-Performance counter measurements can be found [here for Intel](Counters Intel) and [here for AMD](Counters AMD)

The resulting heuristics in use in memutil, as well as discarded ones are explained on the [Heuristics page](Memutil Heuristics).

Memutil general architecture is then explained [here](Memutil Architecture).

We have evaluated memutil on a set of workloads.
The results of this evaluation can be found on the [Evaluation page](Evaluation)

## Running memutil on your machine
Our CpuFreq governor is implemented as a kernel module in the [`kernel-module`](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/tree/master/kernel-module) subfolder of this repository.

To see how to compile & use the kernel module, see the [README in that folder](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/kernel-module/README.md).

memutil will need adjustment for your specific machine for best performance.
The [Porting page](Porting) includes instructions of how to tweak the Heuristic parameters.

## Future Work
See [this page](Future work).

## Other Resources
* Our [project kickoff presentation](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/resources/Initial%20Presentation.pdf) that describes our initial research and project motivation.
