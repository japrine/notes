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

---

<details>
  <summary>File Locations</summary>
  
* Non-ZFS Boot load - /etc/default/grub (afterward update-grub)
* Add Modules - /etc/modules
* Driver loading blacklist - /etc/modprobe.d/blacklist.conf
* VM Config - /etc/pve/qemu-server/<VM-ID>.conf
* Custom romfile location - /user/share/kvm/
* VFIO conf - /etc/modprobe.d/vfio.conf
</details>

---

<details>
  <summary>Install to ZFS soft RAID</summary>
  
* Normal Install - When picking drive, choose option/advance button and choose mirror stripe raidz etc.
</details>

---

<details>
  <summary>IOMMU Hypervisor Setup</summary>
  
* For EXT3/4-LVM GRUB Boot:
  * nano /etc/default/grub
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=1 pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off video=efifb:off"
```
  * update-grub

<details>
  <summary>GRUB_CMDLINE_LINUX_DEFAULT ARGS</summary>
  
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

* For ZFS Boot:
  * nano /etc/kernel/cmdline
```
root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet amd_iommu=on iommu=pt video=vesafb:off video=efifb:off
```
  * pve-efiboot-tool refresh

* nano /etc/modules
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
* reboot

* If needed, blacklist drivers from starting:
```
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```

* If needed, manually set GPU etc to use vfio driver.
  * Check if driver is already vfio using lspci -v
  * To manually set, get device IDs (eg 01:00.0 0000: 10de:1b81 (rev a1)"&" 01:00.1 0000: 10de:10f0 (rev a1) " - You need 10de:1b81 and 10de:10f0.)
  * echo "options vfio-pci ids=10de:DEV_ID1,10de:DEV_ID2" > /etc/modprobe.d/vfio.conf  - replace DEV_ID1&2 with actual addresses from above.
  * After rebooting, changes can be verified with lspci -v

</details>

---

<details>
  <summary>IOMMU VM Setup</summary>
  
* BIOS: OVMF(UEFI)
* Add->EFI Disk
* Machine: G35
* nano /etc/pve/qemu-server/100.conf
```
cpu: host,hidden=1,flags=+pcid
```

* If Video BIOS Needs to be modified to get working:
  * Download BIOS using NVFlash
    * Download NVFlash and create/move to C:\nvflash\
    * Run CMD as Admin and run:
    * nvflash.exe --save gpubios.rom
  * Using Hex Editor (HxD20 Works) open rom:
    * Find "UªyëK7400éLwÌVIDEO" which is the start of the actual bios, delete everything before it.  THe offset for that line should change to 8 zeros.  Save as.
  * Using WinSCP etc, upload modified rom to /user/share/kvm/
  * Rom can be specified in VMID.conf with ',romfile=fixed_gpubios.rom' added to the hostpci0 passthrough line.

* Optional ARGS that sometimes help:
  * args: -cpu 'host,+kvm_pv_unhalt,+kvm_pv_eoi,hv_vendor_id=NV43FIX,kvm=off'

</details>

---

<details>
  <summary>Manual VM Managment Commands</summary>
  
* qm stop VMID - Manually Stop VM
* qm destroy VMID - Delete VM
* qm unlock VMID - Unlock if needed before Destroy etc
</details>

---

<details>
  <summary>Linux / Host Managment Commands</summary>
  
* sgdisk --zap-all <device> - Clears all partitions so disk can be reassigned/used
</details>
