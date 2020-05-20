# Known problems
### The device in RHCOS
The disk for RHCOS should always be stripped of the /dev/ part.
As RedHat is working toward the new installer (the one that FCOS is already using) it will automatically understand the /dev/ pormat (and hopefully support scsi and md device with strange names.)
Variable: 
- `hosts_defaults.disk`

Default Value: 
- `sda`
---

### The network interface
Variable:
- `hosts_defaults.network.interface`

Default Value: 
- As today I automatically switch between `eth0` and `ens192` if the parameter is not hardcoded, in the near future AFAIK this should revert to ens192 in vmware.
---
### The Grub installation drops you in grub shell.

Sometime the lexer cannot interpretate the script at first try.<br>
This will send you to the grub prompt<br>
In this case you need to run the command `reboot` and select boot from CDROM to try again.<br>
For what I've been able to debug is a module of grub that is not too much collaborative.
Help is wanted to solve it.