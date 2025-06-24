WEEKLY REPORT 1
---------------

During this week, according to the proposal:
(x): Complete what is left for FIXME and TODO’s.
(x): Do even more through research on LASI Network Card and NCR710 datasheets,architectural mapping to QEMU’s existing architecture, and potentially create documentation that will help me in the upcoming weeks along with diagrams and
visualisation.
(x): I will do my best to streamline the workflow for upcoming weeks to have a smoother implementation.
(x): Refine the setup of the testing environment.
(x): Discuss with Helge about the plans for upcoming weeks. 
(x): Here we plan to comb through, scrutinize and modify any sections of the current proposal that might require further
adjustments, this should be done to ensure a smooth execution.

This should set the stage for the upcoming weeks

Report:

>> Complete what is left for FIXME and TODO’s.
The fix me and TODO's mentioned throughout the code base are as follows:
SUSPENDED State management:
I have done implmenetation for this although the code is not pushed to upstream(Offical QEMU repo). These implementation work for Debian (HPUX yet to test).

DUMP Command:
This command is a bit complex and requires full implementation of the card including all its functionallity to be dumped, I have implemented the feature of dumping the details which are working as of now in the dump command. (Works on Debian, HPUX yet to test).

Diagnose Command:
This feature is easy to implement for debain as we just send a success code and debain will accept it. However for HPUX its a different story as it performs more through testing.

>> Do even more through research on LASI Network Card and NCR710 datasheets,architectural mapping to QEMU’s existing architecture, and potentially create documentation that will help me in the upcoming weeks along with diagrams and visualisation.

Yes, I have done that although in my physical notebook.

>> I will do my best to streamline the workflow for upcoming weeks to have a smoother implementation

This was done in later weeks as my intial attempt to streamline the workflow with my repo was not upto the mark as I had set my upstream to QEMU unstable branch. Thus changing repo and switching upstream branch to QEMU stable-10.0 and later streamlining the workflow into (experimental, devel and staging branches was a good decision)

>> Refine the setup of the testing environment.
Yes this has also been completed.

>>Discuss with Helge about the plans for upcoming weeks.
I have discussed with Helge the upcoming plans for the weeks and shall follow it accordingly.

>> Here we plan to comb through, scrutinize and modify any sections of the current proposal that might require further adjustments, this should be done to ensure a smooth execution.

I have made a 2.0 version of the proposal with all the changes, as I understood the current proposal was not upto the mark.

Sign-off: Soumyajyotii Ssarkar