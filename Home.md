This is the homepage of the project: "Energy-Aware computing: Cut compiler carbon footprint"

We are focusing on creating a new Linux [CpuFreq Governor](https://www.kernel.org/doc/html/latest/cpu-freq/index.html) that makes dynamic voltage scaling decisions based on hardware performance counters. This will allow it to react to the memory access patterns of the current CPU processes, so that it can reduce clock frequency, should the CPU be stalled by memory.

## Background & Related Work
See [this page](Background-&-Related-Work)

## memutil
Our CpuFreq governor is implemented as a kernel module in the [`kernel-module`](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/tree/master/kernel-module) subfolder of this repository.

To see how to compile & use the kernel module, see the [README in that folder](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/kernel-module/README.md).

The broad architecture of memutil is explained [here](Memutil Architecture).

For a description of the heuristics in memutil, see: [Memutil Heuristics](Memutil Heuristics)

The different Hardware-Performance counter measurements we tested can be found [here for Intel](Counters Intel) and [here for AMD](Counters AMD)

## Future Work
See [this page](Future work).

## Other Resources
* Our [project kickoff presentation](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/resources/Initial%20Presentation.pdf) that describes our initial research and project motivation.
