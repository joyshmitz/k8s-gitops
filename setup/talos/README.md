# talos cluster setup

## nodes

### k8s-0

This is a control plane node that runs via proxmox VM on `nas.home`

- MAC: `BC:24:11:73:58:F0` (hared-coded)
- IP: 10.0.7.50
- CPU: VM - 4 cores
- RAM: VM - 8GB
- Disk: VM - 200GB - `/dev/sda`

```shell
sudo qm create 501 \
  --name k8s-0 \
  --ostype l26 \
  --machine q35 \
  --memory 8192 \
  --cores 4 \
  --cpu host \
  --agent "enabled=1,fstrim_cloned_disks=1" \
  --bios ovmf \
  --net0 "model=virtio,macaddr=BC:24:11:73:58:F0,bridge=br0" \
  --scsihw virtio-scsi-single \
  --efidisk0 "ssdtank-proxmox:1,efitype=4m" \
  --scsi0 "ssdtank-proxmox:200,backup=0,iothread=1,ssd=1,discard=on" \
  --ide2 tank-proxmox:iso/metal-amd64.iso,media=cdrom
```

### k8s-a

This is a worker node that runs via proxmox VM on `nas.home`

- MAC: `BC:24:11:F1:D5:9B` (hard-coded)
- IP: 10.0.7.51
- CPU: VM - 4 cores
- RAM: VM - 32GB
- Disk: VM - 200GB - `/dev/sda`
- Disk: SSD for ceph - 1TB `/dev/sdb` -
  (`/dev/disk/by-id/wwn-0x5002538c000fc6fd` passthrough from the host)
- GPU: iGPU passthrough via Intel GVT-g
  - Because I will probably forget this, the `i915-GVTg_V5_1` mdev device is
    possible to use only because the BIOS setting for the GPU memory were
    increased from the default value to the max value.
  - The benefit of using a 'larger' (smaller suffix number) mdev device is that
    it has more memory/compute capacity than the smaller mdev devices and will
    improve transocindg performance. See
    [here](https://blog.ktz.me/why-i-stopped-using-intel-gvt-g-on-proxmox/) and
    [here](https://github.com/intel/gvt-linux/wiki/GVTg_Setup_Guide#53-create-vgpu-kvmgt-only)
    for more information

```shell
sudo qm create 502 \
  --name k8s-a \
  --ostype l26 \
  --machine q35 \
  --memory 32768 \
  --cores 4 \
  --cpu host \
  --agent "enabled=1,fstrim_cloned_disks=1" \
  --bios ovmf \
  --net0 "model=virtio,macaddr=BC:24:11:F1:D5:9B,bridge=br0" \
  --scsihw virtio-scsi-single \
  --efidisk0 "ssdtank-proxmox:1,efitype=4m" \
  --scsi0 "ssdtank-proxmox:200,backup=0,iothread=1,ssd=1,discard=on" \
  --ide2 tank-proxmox:iso/metal-amd64.iso,media=cdrom \
  --hostpci0 "0000:00:02.0,mdev=i915-GVTg_V5_1,pcie=1" \
  --scsi1 "/dev/disk/by-id/wwn-0x5002538c000fc6fd,backup=0,iothread=1,ssd=1,discard=on"
```

### k8s-b

This is a worker node, bare-metal

- MAC: `00:02:c9:57:5c:a8`
- IP: 10.0.7.52
- CPU: Intel i3-7100 - 4 cores
- RAM: 32GB
- Disk: host - 500GB - `/dev/nvme0n1`
- Disk: SSD for ceph - 2TB `/dev/sda` -
  (`/dev/disk/by-id/wwn-0x5002538c000fc709`)
- GPU: iGPU

### k8s-c

This is a worker node, bare-metal

- MAC: `ac:1f:6b:6f:27:5f`
- IP: 10.0.7.53
- CPU: Intel Xeon D-1518 - 8 cores
- RAM: 64GB
- Disk: host - 1TB - `/dev/nvme0n1`
- Disk: SSD for ceph - 2TB `/dev/sda` -
  (`/dev/disk/by-id/wwn-0x5002538c000cdf0f`)

### k8s-d

This is a worker node, bare-metal - **candidate for retirement**

- MAC: `00:1e:06:45:0b:cd`
- IP: 10.0.7.54
- CPU: Intel J4105 - 4 core
- RAM: 32GB
- Disk: host - 500GB - `/dev/nvme0n1`
- GPU: iGPU

### k8s-e

This is a worker node, bare-metal

- MAC: `00:02:c9:55:b4:26`
- IP: 10.0.7.55
- CPU: AMD Ryzen 3 3200G - 4 core
- RAM: 32GB
- Disk: host - 256GB - `/dev/sdb`
- Disk: SSD for ceph = 2TB - `/dev/sda` - **note this is `/dev/sda`** -
  (`/dev/disk/by-id/wwn-0x5002538c000e0a55`)

### k8s-f

This is a worker node, bare-metal

**2025-05-07:** This node appears to be dead (won't power on)

- MAC: `68:1d:ef:34:d8:1a`
- IP: 10.0.7.56
- CPU: Intel N100 - 4 core
- RAM: 16GB
- Disk: host - 1TB - `/dev/nvme0n1`
- GPU: iGPU

This is a worker node, bare-metal

### k8s-g

This is a worker node, bare-metal

- MAC: `68:1d:ef:34:66:3f`
- IP: 10.0.7.57
- CPU: Intel N100 - 4 core
- RAM: 16GB
- Disk: host - 1TB - `/dev/nvme0n1`
- GPU: iGPU

### k8s-h

This is a worker node, bare-metal

- MAC: `68:1d:ef:34:d9:c7`
- IP: 10.0.7.58
- CPU: Intel N100 - 4 core
- RAM: 16GB
- Disk: host - 1TB - `/dev/nvme0n1`
- GPU: iGPU

### k8s-i

This is a worker node, bare-metal

- MAC: `98:fa:9b:04:47:b9`
- IP: 10.0.7.59
- CPU: Intel i5-8500T - 6 core
- RAM: 16GB
- Disk: host - 128GB - `/dev/nvme0n1`
- Disk: SSD for ceph - 1TB `/dev/sda` -
  (`/dev/disk/by-id/wwn-0x5002538800374ff9`)
- GPU: iGPU UHD 630
