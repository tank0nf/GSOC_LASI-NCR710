WEEK 3 REPORT
-------------

(-): HP-UX loopback testing environment setup and initial testing.
(x): I will add the statistical counters which would be updated to the kernel, this is similar to the approach other NIC takes.
(x): I will completely add the self test feature which makes use of the dump, diagnose, Time Domain Reflectometry and loopback.
(x): I also plan to implement diagnostic commands such as dump internal register, and the
self-test functionality.
(x): Then Command Unit State Machine is missing a few states which would be useful in HPUX but not that much in Debian, so once I get the HPUX working I will add them.

>> HP-UX loopback testing environment setup and initial testing.
I have setup and tested the HPUX 10.20 during its boot process from my analysis and understanding of the working, I have several finding all of which are mentioned in the documentation (HPUX loopback report, please refer to it).
