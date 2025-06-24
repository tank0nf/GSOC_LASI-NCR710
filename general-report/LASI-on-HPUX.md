Getting LASI Working on HP-UX 10.20 – Debug Report
--------------------------------------------------

Overview
--------
After several attempts to get LASI working with HP-UX 10.20 in QEMU, I’ve compiled this report reflecting on the journey, insights, and current status. This is my **third** serious attempt, with a much deeper understanding than the first two. The earlier attempts were fairly primitive, but this one focuses on correctness, debugging, and learning from prior experience.

Setup and Initial Work
----------------------
I started by setting up an HP-UX 10.20 boot environment in QEMU and built on top of the existing LASI implementation.

To accelerate progress, I reused some of the earlier work done by Helge (my mentor), who had previously tried to get LASI working on HP-UX. This helped bring the QEMU system to a common baseline where the NIC was detected but not fully functional.

A issue I found was that the IRQ line for the i82596 NIC wasn’t properly wired from the current LASI implementaion in QEMU. I resolved this so that the NIC can now signal interrupts correctly to the CPU.


Boot Tracing with GDB
---------------------
Using GDB, I traced the HP-UX boot process with a breakpoint on, lasi_chip_write_with_attrs() I found this:

```c
i82596_s_reset 0x555558d0e620 Reset chip
lasi_chip_mem_valid access to addr 0x10 is 1
lasi_chip_read addr 0x10 val 0xfffb0003
lasi_82596_mem_readw addr=0xc val=0xbeefbabe
lasi_chip_write addr 0x4 val=0x00000000
lasi_chip_write addr 0x8 val=0x00000000
lasi_chip_read addr 0x0 val=0x00000000
lasi_chip_write addr 0x10 val=0xfffb0003
lasi_chip_write addr 0x4 val=0x00004000
lasi_chip_write addr 0x4 val=0x00204000
lasi_chip_read addr 0xc004 val=0x00000000
i82596_receive_analysis >>> multicast packet rejected
lasi_chip_write addr 0x4 val=0x00204020
lasi_chip_read addr 0x0 val=0x00000020
(repeated)
```

Reset and Initialization Behavior
---------------------------------
HP-UX resets the NIC early in boot via:
device_class_set_legacy_reset(dc, lasi_82596_reset)
This triggers i82596_s_reset() and initializes internal state.

After reset, HP-UX calls i82596_can_receive() only once.
I modified this function to ensure the NIC is marked "ready to receive" at this early stage. HP-UX does not retry, so this is our only chance to prepare the NIC correctly.

Observations on Memory Access
-----------------------------
A clear pattern emerges:
lasi_chip_write addr 0x4 val=0x00004000
lasi_chip_write addr 0x4 val=0x00204000
lasi_chip_write addr 0x4 val=0x00204020

This likely represents command block (CB), system control block (SCB), or ISCP address writes. HP-UX is trying to communicate with the NIC indirectly through LASI and shared memory region but our implementation of LASI just accepts the addr values and takes it as its own.

Interestingly, HP-UX never writes to 0x7000 (LASI_LAN) or 0x7000 + 12 during boot, which are standard 82596 access points. 
This suggests a different access method via LASI is being used.

Hypothesis: Loopback Timing Issue
---------------------------------
HP-UX sends a loopback packet very early in boot. If QEMU’s NIC isnt ready at that exact moment, the packet is missed.

In regular (non-debug) boot, this causes the NIC to silently fail setup.
In GDB (when paused or delayed), the loopback sometimes succeeds indicating that when tracing through with GDB we give QEMU enough time to initiate the LASI and make it fully functional so it can issue the loopback.

Additional Findings
-------------------
1.Unlike Debian, HP-UX issues probing commands but these commands are being used by LASI not NIC and the values should  commands (e.g, channel attention or CU start). It uses a much more probing-based approach.

2. Command values sent:
lasi_chip_write addr 0x4 val=0x00004000
lasi_chip_write addr 0x4 val=0x00204000
lasi_chip_write addr 0x4 val=0x00204020
These values are likely control structure pointers being forwarded to the NIC. Our current LASI implementation is not processing them correctly, so HPUX cannot fully initialize the NIC.
Thus when the HPUX issues the loopback packet, it gets dropped silently.