# Table of comparision
| Name  | Require OVA| Require DHCP    | Require WebServer  | Need ISO |  Support FCOS | Support RHCOS | Support Baremetal | Description  |
|---|---|---|---|---|---|---|---|---|
| grub   |  ✅ | 🚫  |  ✅  | ✅| ✅  | ✅  | 🚫 | I will generate a iPXE iso and chainload once a grub pxe to inject IP parameters. |
| template   |  ✅ | ✅*  | 🚫  |🚫  | ✅  | ✅  | 🚫 | I will generate and inject an ignition and use the ova template for vmware installation The DHCP is needed for the image to start but the injected parameters in ignition will fix them at first reboot. |
| netinstall   | 🚫|  🚫|  ✅  |  ✅   | ✅  | ✅| 🚫 | I will install the OS via network and inject the ip via kernel parameters |

*In the template mode you need dhcp at first boot because of a bug in coreos/rhcos <br>See [GitHub Issue](https://github.com/coreos/fedora-coreos-tracker/issues/358) and [Fedora COS Docs](https://docs.fedoraproject.org/en-US/fedora-coreos/static-ip-config/)

# Methods description and **backdraft**
## Method Grub:
### PRO: 
- You don't need to set up DHCP reservation
- You don't need to setup transpile the installer detect the parameter at boot and staticize it.
- Don't leave *trace* in the installation as it just compile for you the parameter for the kernel to stacizie the IP.
- You are using an OVA so you don't need to download a ton of files.

### CONS:
- You need a webserver to chainload a grub
- You need to boot from an ISO to chainload Grub
- This method is still not supported by redhat. (But at least you don't leave any trace).
- Still buggy, sometime the lexer cannot interpretate the script at first try.<br>
This will send you to the grub prompt<br>
In this case you need to run the command `reboot` and select boot from CDROM to try again.

------------

## Method Template:

### PRO: 
- You don't need to set up DHCP reservation
- You don't need a webserver to host PXE informationss
- You are using an OVA so you don't need to download a ton of files.

### CONS:
- You need to transpile the network configurations
- Red Hat does not document in any way that transpiling desn't broke your support.
------------

## Method netinstall:

### PRO:
- You don't need to set up DHCP reservation
- You don't need to transpile the network confiuguration

### CONS:
- You need a webserver to host the ignition configuration
- You need an ISO to start the PXE installation
- Red Hat does specify that the installation on vmware is done via OVA (AFAIK does not change anything installing via network or OVA in the final image).
