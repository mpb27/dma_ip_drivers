# Xilinx XDMA IP Reference drivers

Fork of `sasfermat/dma_ip_drivers`.  Simplying code by removing features, and cleaning it up at the same time.

### TODO

- No polling mode.
- No performance stuff.
- Remove splitting descriptors that have a transfer size larger than 256MB. This can be enforced at a higher level.
- Remove schedule_work and do most of the work in the interrupt or a tasklet.
- I'm torn about cyclic mode, because in my application I can create a pool of transfers rather than using a cyclic mode. The pool of transfers can be backed by user space memory using either ioctl or async reads.

