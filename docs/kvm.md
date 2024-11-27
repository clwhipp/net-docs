# Virtual Machines

## Creation

Creation of virtual machines handled through virt-install command. The virt-install command
is installed as part of the virt-manager package. The following is an example of creating a
virtual machine.

```bash
virt-install \
     --connect qemu:///system \
     --virt-type kvm \
     --name ns1 \
     --vcpus 1 \
     --ram 1024 \
     --disk path=/mnt/internal/vm_store/disks/ns1-disk.img,size=20 \
     --network bridge=br0 \
     --graphics vnc,listen=0.0.0.0 \
     --cdrom /mnt/internal/vm_store/iso/ubuntu-22.04.1-live-server-amd64.iso \
     --os-variant ubuntu21.04
```

The previous command will start the virtual machine and create a VNC server on port 5900 (default)
for interacting with machines console. Downloading VNC client and pointing to the host on port 5900
will provide access to an interactive console.

The creation process is ran by libvirt-qemu user within the system. As such, the iso files and locations
for storing the disks must be accessible by libvirt-qemu. Permissions errors will result in failure
to create the virtual machine.

```bash
root@vcenter:/etc/netplan# ls -lah /mnt/internal/vm_store/
total 3.4M
drwxrwx--- 4 libvirt-qemu kvm 4.0K Nov  4 02:57 .
drwxr-xr-x 4 cameron      kvm 4.0K Nov  4 02:30 ..
drwxrwx--- 2 libvirt-qemu kvm 4.0K Nov  5 21:58 disks
drwxrwx--- 2 libvirt-qemu kvm 4.0K Nov  4 02:11 iso
-rwxrwx--- 1 libvirt-qemu kvm  21G Nov  4 02:50 ubuntu22.04
```

## Deletion

## Start/Stop/Reboot Machine

Machines that are running can be listed using the virsh list command.

```bash
root@vcenter:/etc/netplan# virsh list --all
 Id   Name   State
----------------------
 11   ns1    running
 ```

Virtual machines can be stopped using the virsh poweroff command when desired.

```bash
virsh poweroff ns1
```

Machines can be started with virsh start

```bash
virsh start ns1
```

## Shared Directories

Sharing directories between the host and guests will be necessary to support RAID
storage. The following needs to be added to the virt-install command when the machine
is created to support shared directories.

```bash
     --filesystem /mnt/critical/source,/sharename
```

With the fileystem argument added the command would look like the following:

```bash
virt-install \
     --connect qemu:///system \
     --virt-type kvm \
     --name ns1 \
     --vcpus 1 \
     --ram 1024 \
     --disk path=/mnt/internal/vm_store/disks/ns1-disk.img,size=20 \
     --network bridge=br0 \
     --graphics vnc,listen=0.0.0.0 \
     --filesystem /mnt/local/path,/sharename \
     --cdrom /mnt/internal/vm_store/iso/ubuntu-22.04.1-live-server-amd64.iso \
     --os-variant ubuntu21.04
```

This will create a share within the guest called "sharename" that can then be mounted
within the guest VM. There are two methods by which the directory can be mounted inside
the VM.

First, the share can be mounted manually with the following commands.

```bash
mkdir -p /path/in/host
sudo mount -t 9p -o trans=virtio /sharename /path/in/host
```

The second option is to update fstab so that it automatically mounts each time the VM
starts. The following line could be added to the /etc/fstab to support automated mounting.

```
/sharename /path/in/host 9p trans=virtio,version=9p2000.L,rw 0 0
```

This will create a mount within the guest that allows for saving files that will become available
within the host.