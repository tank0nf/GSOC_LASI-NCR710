Getting LASI Working on HP-UX 10.20 – Debug Report(Revised)
-----------------------------------------------------------

Initially I had a hunch that their was an issue with our LASI implementation. But then when testing further with ODE the below mentioned was clear:

Okay so, from the HPUX ODE we can confirm that HPUX can properly communicate with LASI 82596 Chip. As in the logs stated below:

lasi_82596_mem_writew addr=0x0 val=0x0001
i82596_s_reset 0x555556a4eb00 Reset chip
lasi_82596_mem_writew addr=0x4 val=0xb4b1
lasi_82596_mem_writew addr=0x4 val=0x0018
lasi_82596_mem_writew addr=0x8 val=0x0001

I was intially skeptical of the ODE as i used the following commands:

ISL> ODE
ISL>LASIDIAG

This showed us a screen where it is visible that LAN has "Y#" This means, yes HPUX is sensing our 82596 but the only issue is 

"#": No loopback device present

This I had initally assumed as the ODE did not have tests for Loopback, but then I realised the "#" is most probably reffering to a connection between  the TX and RX unit which is not being sensed by HPUX although we have a clear connection between the TX and RX function throught the Loopback Mode in our code.

This is farely interesting, now coming back to our implementation.

As I started the self test module for HPUX LAN in the ODE it is evident from the logs that HPUX does access the lasi_82596 glue to write commands and its working as intended. But the only issue is its not the same during boot.

Testing with tulip implementation on HPUX although their is loopback code in RX it is never triggered during boot.

Now during tulip boot the initial logs are of:
reset and then, tulip write and read registers which trigger the tulip_xmit_list_update() and the rest of it, however for our i82596 the i82596_mem_write and read in our lasi_82596 glue are never triggered (in normal boot, although they are triggered when testing with ODE).

This is puzzling, when comparing with the tulip boot process the device is reset and then it triggers the normal, read and write registers for tulip, but not the same for LASI?

So I deduced that maybe the kernel is not ready to call the lasi periferals yet, so I added a "sleep(90);" in the lasi_chip_write_with_attrs() function's LASI_IMR as it was the ones triggered the least during write thus this would slow down our boot process enough to maybe the kernel could get to its full functioning state, however this proved that.

i82596_s_reset 0x555558d0e620 Reset chip
i82596_s_reset 0x555558999500 Reset chip
lasi_chip_mem_valid access to addr 0x10 is 1
lasi_chip_read addr 0x10 val 0xfffb0003
lasi_82596_mem_readw addr=0xc val=0xbeefbabe
lasi_chip_mem_valid access to addr 0x4 is 1
lasi_chip_write addr 0x4 val 0x00000000
lasi_chip_mem_valid access to addr 0x8 is 1
lasi_chip_write addr 0x8 val 0x00000000
lasi_chip_mem_valid access to addr 0x0 is 1
lasi_chip_read addr 0x0 val 0x00000000
lasi_chip_mem_valid access to addr 0x10 is 1
lasi_chip_write addr 0x10 val 0xfffb0003
lasi_chip_mem_valid access to addr 0x4 is 1
lasi_chip_write addr 0x4 val 0x00004000
lasi_chip_mem_valid access to addr 0x4 is 1
lasi_chip_write addr 0x4 val 0x00204000
lasi_chip_mem_valid access to addr 0xc004 is 1
lasi_chip_read addr 0xc004 val 0x00000000
i82596_receive_analysis >>> multicast packet rejected
lasi_chip_mem_valid access to addr 0x4 is 1
lasi_chip_write addr 0x4 val 0x00204020
lasi_chip_mem_valid access to addr 0x0 is 1


i82596_receive_analysis >>> multicast packet rejected
However, now we see that HPUX is directly sending the packet. Without going through the LASI glue to write commands first or setting up tht CU and RU state and then configuring the i82596 to loopback mode and then sending the packet, so the packet gets rejected which is obvious.

This lead me to look into the inital probing of i82596 that HPUX does.

During initial probing of i82596 the device is first reset then through lasi_82596_realize()->net_lasi_82596_info's mem region i82596_can_receive() is triggered which checks the i82596 to see if it can recieve the packet. This is done in the very early stages of boot, so i modified the function to return true.

Now it occured to me that maybe HPUX is sending a scattered packet, so i implemented the iov() function for i82596, which to my suprise is what was triggered with the sleep timer rather than the normal receive function.

Then looking into the NetClientInfo stuct i found the "NetStart *start;" this was promising so i hard coded it to set the device into a specific state clearing SCB and CU and RU setting the 82596 into loopback mode, but as of now im doing it yet it hasnt proven effective yet.

If this doesnt work, pretty not sure what could be next


LASI DIAG also giving some bus error but i dont think that is relevant to our current lasi 82596 implementation.

Honourable Mentions:
Out of plans, I also tried out the hackish way that was to accept the first packet regardless of the circumstances. This also didnt seem to work, so their is probably more to the story that HPUX expects.