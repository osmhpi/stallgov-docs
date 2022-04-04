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
    - Of the CPUs availabe to us, currently fast-switching is only available on the recent Intel CPUs
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

**Note:** We set the frequency every time in the update hook, even if we set it to the same frequency we had before. We do this to simplify the code as otherwise we would need to check and track various reasons an update might be needed which would be:
 1. We, ourself, want a different frequency because performance counter values changed.
 2. The limits (max- and min-frequency) changed so we have to set a new frequency (has to be inside the limits).
 3. The driver wants a frequency update.

The update hook does a few checks to make sure the frequency for the current core can actually be set.
Then it calls the `memutil_update_frequency` method, which reads the performance counters, chooses an appropriate frequency and then sets the frequency.
This is done either by using a fast-switch call, or by storing the requested frequency so it can be updated later by the kernel thread responsible for slow switching the frequency.
For an explanation of how the frequency is chosen, see the [Heuristics page](Memutil Heuristics).

## stop
The `stop` method basically undoes everything done in the `start` method, de-allocating the performance counters, etc.

## exit
Exit then frees any resources allocated in the `init` method.

## Note on policies vs cores:

The frequency governors work with policies: One policy defines how the frequency is managed (governor to use etc.) an can be assigned to one or multiple cores. In case it is used for more than one core it is called a shared policy. Reversely there is a mapping of cpu core to the policy that manages this core.
The governors init, start, stop, exit and limits methods are called per policy and we set the frequency per policy in memutil. The update hook called by the scheduler is always called per core and not per policy.

**However** for all of our tested distributions / platforms there was always a one-to-one mapping of policy to cpu core. I.e. there was always one policy per core and shared policies were not used. Therefore our code only works with non-shared policies and we do not need a special distinction of per-core or per-policy as both are the same.

# Logging

To log key data (requested frequency, performance counter values) with each frequency update we had to create some logging functionality. Writing it into the kernel log is not an option because it would be way to much information that would flood the kernel log that is intended for other things.
Logging happens in two "phases" in the kernel module and one phase outside the kernel module:

## 1. Ringbuffer

Here the data is written as binary data into a ringbuffer. This means that, if the buffer reaches its end, the buffer will wrap around to the start so that new data will start to override the oldest data. This buffer is intended to be fast, so that data can be logged with every frequency update. That leads to this buffer being small. Additionaly we use one ringbuffer per cpufreq policy (i.e. per cpu-core) which reduces contention. The size can be configured with the default being a size of 2000 elements. With that setting and one frequency update every 5ms this buffer can store data for 10 seconds.

## 2. Logfile

To allow the user to access the data for all cores in textual form we implemented a logfile in the [debugfs](https://www.kernel.org/doc/html/latest/filesystems/debugfs.html) located under `<debugfs>/memutil/log`. This is a virtual filesystem, i.e. the files do not exist in the classical way on a harddrive. Instead functions inside the kernel are called when the user accesses these files. That means for our logfile a read method (`user_read_log` in `memutil_debugfs_logfile.c`) is called when the user wants to read the log. Only when this method is called the ringbuffer data across all cores is collected, converted into a textual representation and provided to the user. For this purpose a second, larger buffer exists that stores this converted textual representation. Also when the user reads the log, the log is cleared afterwards. This way no extra way of clearing the buffers is needed, instead it is already integrated into the reading process. 

## 3. Userspace copy

As the buffers in the kernel only store data for some seconds a userspace script (`copy-log.sh`) is used to copy the log every few seconds and append its data to a persistent logfile somewhere on the harddrive. As the copy also clears the kernel space buffers this way we persist the log and make space for new data to be logged.
The script not only persists the joined log but also splits it into one log per core to allow easier processing later on. The period with which the log is copied as well as the amount of cores into which the log has to be split can be specified as parameters to the script. The time until the ringbuffer is full is written into the kernel log. Also there exists an infofile under `<debugfs>/memutil/info` that contains the frequency update interval as well as the ringbuffer size. With that the time until the ringbuffer is full can be calculated as:
```math
u = update\_time_{seconds}\\
s = ringbuffer\_size\\
e = elements\_per\_second\\[14pt]
e = \frac{1}{u}\\[14pt]
time_{full} = \frac{s}{e} = \frac{s}{\frac{1}{u}} = s \cdot u\\[14pt]
\text{Default:   } time_{full} = 2000 \cdot 0.005s = 10s
```
