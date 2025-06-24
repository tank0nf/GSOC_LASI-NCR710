WEEK 2 REPORT
-------------

During this week the plan was as follows:
(x): I plan to implement the Rx and Tx function working properly.
(x): Then implement the Command Unit State Machine with proper transition
(x):I have noticed a few issues with memory structure addressing for both endian modes, so I plan to implement alignment handling and address endianness conversion for PA-RISC.
(-):Update the QEMU Mailing List with the current progress and code for feedback. (Update:1/6)


>> I plan to implement the Rx and Tx function working properly.
I have implemented clean, accurate to the documentation models for the Rx and Tx functionality of 82596CA. Although the issue of the current implementation having overruns is one that prevails.
This could be easily fixed for later by adding a check beforehand for the RFD buffers.
I plan to keep this issue now as it will be Helpful with the Statistical counters implementation Testing.

>> Then implement the Command Unit State Machine with proper transition.
This implementation has been done before, the debain OS doesnt require the transition states. But the HPUX requires them so getting our NIC working on HPUX is one of the main issues that we are seeing arise again and again.

>> I have noticed a few issues with memory structure addressing for both endian modes, so I plan to implement alignment handling and address endianness conversion for PA-RISC.
I have a few test code implementing the device endianess the debian uses big endian mode only. But the main testing should be done with HPUX which still remains an issue to resolve.


Additionally, I have done some testing with the HPUX 10.20 and identified some issues which are yet to be resolved (Please refer to the HPUX report in general-report). Considering all this the getting our LASI and 82596CA implementation working is still pending.