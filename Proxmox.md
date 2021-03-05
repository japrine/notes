## Proxmox Notes

<details>
  <summary>New Install Stuff</summary>
  
* Disable subscription nag popups
```
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```
* Disable Enterprise Repositories
```
sed -i.bak 's|deb https://enterprise.proxmox.com/debian jessie pve-enterprise|\# deb https://enterprise.proxmox.com/debian jessie pve-enterprise|' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian jessie pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list
```
</details>


<details>
  <summary>File Locations</summary>
  
* Non-ZFS Boot load - /etc/default/grub (afterward update-grub)
* Add Modules - /etc/modules
* Driver loading blacklist - /etc/modprobe.d/blacklist.conf
* VM Config - /etc/pve/qemu-server/<VM-ID>.conf
* Custom romfile location - /user/share/kvm/
* VFIO conf - /etc/modprobe.d/vfio.conf
</details>


<details>
  <summary>Install to ZFS soft RAID</summary>
  
* Normal Install - When picking drive, choose option/advance button and choose mirror stripe raidz etc.
</details>


<details>
  <summary>IOMMU Hypervisor Setup</summary>
  
* nano /etc/default/grub
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=1 pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```
<details>
  <summary>Arg Details</summary>
  
* quiet - non-verbose boot (hides tons of loading and checks)
* pcie_acs_override (Shouldn't be used unless needed for group isolation)
  * downstream - Hack to split IOMMU groups further.
  * multifunction - Further splits Multifunc devices.
* The following are all ways to disable the boot frame buffer (one or more can be used)
  * vga=normal - Disable Frame Buffer
  * nofb - No Frame Buffer
  * nomodeset - Tells Kernel not to load video drivers and use BIOS mode during boot
  * video=vesafb:off - Frame Buffer Off
  * video=efifb:off - UEFI Frame Buffer mapping
  * i915.modset=0 - Frame Buffer Off
Verify if Framebuffer is being used:
```
ls -l /dev/fb*
```
If the frame buffer is enabled, the above command will usually return /dev/fbX (X being a number; usually 0).
or
```
grep -i "frame buffer" /var/log/syslog
```
If the frame buffer is enabled, it should return something such as: "Console: switching to colour frame buffer device 160x64, fb0: inteld"

</details>

* update-grub
* nano /etc/modules
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
* reboot
</details>


<details>
  <summary>IOMMU VM Setup</summary>
  
* BIOS: OVMF(UEFI)
* Add->EFI Disk
* Machine: G35
* nano /etc/pve/qemu-server/100.conf
```
cpu: host,hidden=1,flags=+pcid
```
</details>

