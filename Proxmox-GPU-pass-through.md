# Proxmox GPU Pass through

## Enable the IOMMU

1. Edit the `/etc/default/grub` and update the following line
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
2. Update the `grub`
```bash
update-grub
```

## Add required kernel modules
```bash
cat <<EOF >> /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
EOF
```

## IOMMU interrupt remapping
```bash
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```


## Blacklisting Drivers
```bash
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```

## Adding GPU to VFIO

1. Get the device ID mapping from `lspcie`
```bash
lspci | grep NVIDIA

02:00.0 VGA compatible controller: NVIDIA Corporation TU107 (rev a1)
02:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)
```
> Device ID is `02:00` in this instance

2. Get the PCI device codes using the ID above
```bash
lspci -n -s <ID HERE>

lspci -n -s 02:00
02:00.0 0300: 10de:1f82 (rev a1)
02:00.1 0403: 10de:10fa (rev a1)
```
> Vendor ID codes in this case are `10de:1f82` and `10de:10fa`

3. Add GPU's vendor id's to the VFIO
Add following option to `"options vfio-pci ids=<VENDOR-ID-1>,<VENDOR-ID-2> disable_vga=1"` to the `/etc/modprobe.d/vfio.conf`.

    Replace the `<VENDOR-ID-1>` and `<VENDOR-ID-2>` with values above.
```bash
echo "options vfio-pci ids=10de:1f82,10de:10fa disable_vga=1"> /etc/modprobe.d/vfio.conf
```

4. Generate an initramfs image
```bash
update-initramfs -u
```

5. Reboot the Proxmox server
```bash
rest
```
