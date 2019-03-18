# Early-PCI-driver-unbinding
unbind devices from the kernel early in boot


# Early PCI driver unbinding

Passing through PCI devices is relatively easy, as long as the drivers are modules in the kernel, as the black list process will work.
When the PCI device driver is linked as part of the kernel, the blacklist process will not work.
To fix this you need to unbind the device from the kernel driver early in the boot process.

## Why

An example of this, is new Nvidia cards have four devices that need to be ignored by the kernel

    01:00.0 VGA compatible controller: NVIDIA Corporation Device 1eb1 (rev a1)
    01:00.1 Audio device: NVIDIA Corporation Device 10f8 (rev a1)
    01:00.2 USB controller: NVIDIA Corporation Device 1ad8 (rev a1)
    01:00.3 Serial bus controller [0c80]: NVIDIA Corporation Device 1ad9 (rev a1)

The VGA and audio can be unbound via vfio driver, and blacklisting.

see [https://askubuntu.com/questions/110341/how-to-blacklist-kernel-modules](https://askubuntu.com/questions/110341/how-to-blacklist-kernel-modules)

The Serial bus controller can be done in the same way.

The USB contoller is more difficult as xhci_hcd is built into the kernel, in ubuntu 18.04.

Options are rebuild the kernel, with this as module or unbind it as we boot before we build the device tree

This is unbinding during boot approach

# How

see [man initramfs-tools](http://manpages.ubuntu.com/manpages/xenial/man8/initramfs-tools.8.html)

Note: init-top the scripts in this directory are the first scripts to be  executed  after sysfs  and procfs have been mounted.  It also runs the udev hook for populating the /dev tree (udev will keep running until init-bottom).

You will need to put the following files in these locations
 
    /etc/
    unbindpci.conf

    /etc/initramfs-tools/scripts/init-top
    unbind-early-pci
 
    /etc/initramfs-tools/hooks
    unbind-early-pci-hook
 
Then rebuild initramfs to make active

    sudo update-initramfs -u
 
## unbindpci.conf

This file can have one or more items as follows PCIid,driver name to unbind
You can find the id by lscpi -D (show PCI domain) example

    lspci -D

    0000:01:00.0 VGA compatible controller: NVIDIA Corporation Device 1eb1 (rev a1)1122.xml
    0000:01:00.1 Audio device: NVIDIA Corporation Device 10f8 (rev a1)
    0000:01:00.2 USB controller: NVIDIA Corporation Device 1ad8 (rev a1)
    0000:01:00.3 Serial bus controller [0c80]: NVIDIA Corporation Device 1ad9 (rev a1)

e.g.

    0000:01:00.2,xhci_hcd
    

## unbind-early-pci

Loops through the unbindpci.conf file unbinding the device from the driver.

## unbind-early-pci-hook

Copies the conf file into the initramfs when built so we have access at boot time.

# Debug

lsinitramfs /boot/<image_name> and check scripts are in there

# thank you

based off [https://superuser.com/questions/541854/disable-specific-pci-device-at-boot](https://superuser.com/questions/541854/disable-specific-pci-device-at-boot)