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

### New Install Stuff

* Disable subscription nag popups
```
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```
* Disable Enterprise Repositories
```
sed -i.bak 's|deb https://enterprise.proxmox.com/debian jessie pve-enterprise|\# deb https://enterprise.proxmox.com/debian jessie pve-enterprise|' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian jessie pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list
```

### File Locations

* Non-ZFS Boot load - /etc/default/grub (afterward update-grub)
* Add Modules - /etc/modules
* Driver loading blacklist - /etc/modprobe.d/blacklist.conf
* VM Config - /etc/pve/qemu-server/<VM-ID>.conf
* Custom romfile location - /user/share/kvm/
* VFIO conf - /etc/modprobe.d/vfio.conf

### Install to ZFS soft RAID

Normal Install - When picking drive, choose option/advance button and choose mirror stripe raidz etc.

### IOMMU Hypervisor Setup

* nano /etc/default/grub
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=1 pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```
* update-grub
* nano /etc/modules
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
* reboot

### IOMMU VM Setup

* BIOS: OVMF(UEFI)
* Add->EFI Disk
* Machine: G35
* nano /etc/pve/qemu-server/100.conf
```
cpu: host,hidden=1,flags=+pcid
```

