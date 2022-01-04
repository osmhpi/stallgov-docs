# Prerequsites

Unfortunately, as of the time of writing, you need a patched kernel.

See \[this page\](<https://kernelnewbies.org/KernelBuild>) on how to compile & install your own kernel.

For fedora, I recommend you follow \[this guide\](<https://fedoraproject.org/wiki/Building_a_custom_kernel#Building_a_kernel_from_the_exploded_git_trees>) and use the instructions under the "Building a kernel from the exploded git trees" section.

Then add this line:

```
EXPORT_SYMBOL_GPL(perf_event_read_local);
```

After the definition of `perf_event_read_local` in `kernel/events/core.c` .

At last, compile and install the new kernel.

# Dependencies