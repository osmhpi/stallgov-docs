# CpuFreq Basics
The Linux kernel needs to make sure the CPU(s) it is running on are used to the best of their performance in terms of performance and power-consumption.
For this reason, it includes a CPU Performance Scaling mechanism called CpuFreq that allows the CpuFreq governors to switch the performance/power state of the CPU.
Some good resources to get started on this topic can be found here:
- https://www.kernel.org/doc/Documentation/cpu-freq/governors.txt
- https://www.kernel.org/doc/html/v4.14/admin-guide/pm/cpufreq.html

As mentioned in both articles, the most current approach implemented in the kernel is based on the Per-Entity-Load-Tracking (PELT) and implemented in the "schedutil" governor.
That does however mean schedutil only takes the load characteristic of the active processes into account.
This results in schedutil choosing the maximum available frequency when the system is under full load.
As elaborated on in the [Background & Related Work page](Background-&-Related-Work), research suggests that Dynamic Voltage and Frequency scaling can still save energy, even when the system is under full load.
A lot of the work may be bound by external resources like memory, which means that increasing the CPU frequency doesn't benefit the overall performance and only wastes energy.

Therefore we aim to create a new CpuFreq governor based on the workloads memory characteristics that can provide reduced energy consumption at full load.
We call this governor "memutil" - short for memory utilization - similar to the naming scheme of schedutil.

## CpuFreq Governor as kernel module
Cpufreq Governors - apart from "userspace" - are run in kernel space.
The existing governors are all implemented directly in the kernel.
Modifying these governors requires recompiling and installing a new kernel every time.
As this would slow down development a lot, we opted to implement our governor as a kernel module instead.

Kernel modules are basically shared libraries that the kernel can load and unload at runtime.
For CpuFreq governors, the Linux module system provides special methods to (un-)load the governor that is different from normal kernel (un-)loading.
It is split into 4 different methods:
- init (memutil_init)
- start (memutil_start)
- stop (memutil_stop)
- exit (memutil_exit)

There also exists a fifth method (memutil_limits) that is called whenever the limits of the governor change (i.e. available min/max frequency).
However, we mostly ignore this at the moment.

For a guide on building the memutil kernel module, see the [README in the kernel-module folder](https://gitlab.hpi.de/osm/osm-energy/masterprojekt-ws21-compendium/-/blob/master/kernel-module/README.md).

After building, simply inserting the kernel module won't call any of the aforementioned methods yet.
They will only be called if the CpuFreq Governor is actually selected by the `cpupower frequency-set -g` shell command.
At that point, first memutil_init, then memutil_start will be called for all cores (including virtual cores).

### init method
The init method is called to initialize the policy for the cpufreq governor.
A cpufreq_policy is provided by the kernel and contains the information needed by the cpufreq governor in order to keep within allowed parameters, i.e. min and max frequencies, etc.
There is also a free field in the cpufreq_policy struct for governor data.

In the init method, this field is populated with a pointer to a governor-defined struct `memutil_policy` that contains any additional data memutil needs to make decisions.

Additionally the init method also asserts whether the CPU supports fast-switching the power state.

There are 3 ways a CpuFreq governor can change the power state
- Fast-switching frequency
    - The governor chooses a frequency in KHz and instructs the CPU to change to it
    - Setting the frequency is done without locking
    - Currently fast-switching only seems to be available on recent Intel CPUs
- "Normal" frequency switching
    - The governor chooses a frequency in KHz and instructs the CPU to change to it
    - Setting the frequency needs locking and is therefore done in a separate thread
    - This method seems to be available on both Intel and AMD CPUs
- P-State switching
    - TODO