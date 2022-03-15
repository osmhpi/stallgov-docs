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

## init method
The init method is called to initialize the policy for the cpufreq governor.
A cpufreq_policy is provided by the kernel and contains the information needed by the cpufreq governor in order to keep within allowed parameters, i.e. min and max frequencies, etc.
There is also a free field in the cpufreq_policy struct for governor data.

In the init method, this field is populated with a pointer to a governor-defined struct `memutil_policy` that contains any additional data memutil needs to make decisions.

The init method also allocates a worker thread if fast-switching is not supported.
This worker thread sets the actual CPU frequency, if only the default slow switching method is available.

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
    - This method seems to be the newest of the three
    - P-state uses the notion of "Power States" in comparison to the traditional frequencies.
    - With P-state, the CpuFreq Governor describes "how much of the available performance" is currently required.\
        For this it uses an abstract scale bound by two integer values.\
        The CpuFreq governor can then choose any number within that range, which describes how much performance is needed.\
        This information is then processed by the processor driver, which chooses an appropriate voltage and frequency pair, depending on the required performance.
    - We currently do not support this way of setting the frequencies, as it was only available with the Intel P-State driver at the time of writing.\
        AMD is currently working on providing a P-State driver of their own.
    - This method of setting the frequency might result in better control for the CpuFreq driver, as there is the possibility that it might influence Turbo-Boost behavior as well, however we haven't tested this yet.

## start method
The `start` method is called after the init method for each core that is switched to the memutil governor.

It is responsible for setting up the policy data, performance counters, starting logging and installing the memutil update hook.

### Performance counters
The performance counters required by memutil are allocated in the `memutil_start` method and de-allocated in the `memutil_stop` method.

The linux kernel has two types of available events. 
- "Named events"
    - Are common to most architectures (like number of elapsed cycles).
    - There are only few of these and are therefore relatively limiting.
    - Documentation on these can be found in the documentation of the `config` variable in the [`perf_event_open` documentation](https://man7.org/linux/man-pages/man2/perf_event_open.2.html).
- "Raw events"
    - Each CPU has its own performance counters, that can be accessed by writing a custom binary value into the `config` field of the `perf_event_open` call.
    - The difficulty with these events is determining which binary values to set them to.
    - perf has a giant table of the event codes for each supported hardware platform. We use this table in a simplified form (REGEX is not supported in the kernel) in the pmu_events.c/.h files.

As `perf_event_open` is a syscall, we cannot call it directly from within the kernel, but use the kernel-internal methods.
These have already been used by Intel engineers for a similar purpose, as can be seen [here](https://lwn.net/Articles/354497/).
For reading the performance counters we must also use the `memutil_perf_event_read_local` method.
This method is a duplicate of an internal kernel method, which unfortunately isn't exported to kernel modules.
It is important that it is the `_local` read call, as the non-local read will cause the kernel to deadlock.
 
### Update hook - `memutil_update_single_frequency`
The update hook is a method that the CpuFreq governor registers with the kernel.
It will be called by the kernel periodically which allows the governor to change the frequency at that point.

In memutil, the update hook is the `memutil_update_single_frequency` method.
It updates the CPU frequency every time it is called.

**Note:** The update hook **must** set a frequency whenever it can. If too many invokations of the update hook took place without a frequency update, the CPU driver will assume the governor unresponsive and start adjusting the frequency of its own volition!

The update hook does a few checks to make sure the frequency for the current core can actually be set.
Then it calls the `memutil_update_frequency` method, which reads the performance counters, chooses an appropriate frequency and then sets the frequency.
This is done either by using a fast-switch call, or by storing the requested frequency so it can be updated later by the kernel thread responsible for slow switching the frequency.

## stop
The `stop` method basically undoes everything done in the `start` method, de-allocating the performance counters, etc.

## exit
Exit then frees any resources allocated in the `init` method.