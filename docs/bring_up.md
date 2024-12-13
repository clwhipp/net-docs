# Server Installation

The following page describes the process for recreating a server
with the documented architecture using the tools configured within
the environment. The process of rebuilding the server will change
based on if a lighthouse or the primary container host is being
rebuilt.

In general, the process can be outlined as follows:

1. Install OS - Typically a Ubuntu Server OS (LTS Edition)
2. Ansible Host - Prepare the ansible host to support rebuild
3. Execute Playbook - Run pre-built playbook on new server
4. File-system Encryption - Login and enable file-system level protections
5. Enable Nebula - Login to the server and enable nebula connectivity service
6. Unlock RAIDs - RAIDs will need to be unlocked to provide services with data
7. Start Services - Login to the server and start appropriate containerized services
8. Update DNS (if necessary) - Update DNS records if necessary for clients
9. Borg Backup Keys - Install wrapped borg keys on system so that automated backups are possible


## Install OS

As of now, the choice for servers is Ubuntu Server LTS. At the time of this writing, the
version that was installed with 24.04 LTS. The LTS versions were chosen for their
extended period of support.

The ISOs can be downloaded from [here](https://ubuntu.com/download/server). Then, the
ISO should be installed on a bootable USB to support load into the system. The process
will obviously be different than this for cases where cloud hosted services
or virtual machines are being utilized.

The following settings should be utilized during installation

1. Minimized Install
2. Enable OpenSSH
3. Memorable Username and Password
4. Enable Full Disk Encryption (Optional)

Once the installation has completed, make note of the ip address of the machine. This will be needed
during next step for ansible.

```bash
ip addr
```

## Execute Playbook

Once OS is installed, the machine is ready to be bootstrapped and subsequently configured. This
configuration is completed through a series of ansible playbooks that are resident [here](https://github.com/clwhipp/ansible).

The first likely step is that the host variable files are likely not correct from perspective
of the IP address. The **ansible_host** variable needs to be updated within the appropriate
host variable file. This enables ansible to find the machine it's configuring.

Next, the desired playbook for setting up the machine needs to be validated to ensure that
it's looking at the appropriate host. For this walkthrough, I validated that [setup-all-in-one.yaml](https://github.com/clwhipp/ansible/blob/main/setup-all-in-one.yml)
is indeed pointing at the appropriate host. If the hosts is incorrect, the changes will be made
to the incorrect machine.

```yaml
- hosts: nexus
  become: true
```

Next, the Bitwarden CLI account needs to be unlocked to make secrets available
to ansible. This can be done using [ykbw](https://github.com/clwhipp/yubikey-utils/blob/main/ykbw)
which is a custom built python application to enable unlocking a bitwarden
vault using a yubikey with hmac-sha1.

```bash
ykbw unlock
```

When prompted, tap the yubikey to allow use of it's secret for unlocking the vault. The success
of the previous unlock can be verified with **bw status**. With the vault unlocked, its now
time to kick off the bootstrapping process.

The bootstrapping process will run the server through some preliminary steps to get it
in state where the more in depth playbooks can be ran. For instance, it will go through
and setup SSH keys, reset account passwords, and many other things.

The following command will kick off the bootstrap process and then run the specified
playbook. The script will prompt for the current machine password as a means to login
and get it changed.

```bash
./bootstrap-host.sh <name of playbook> <machine>

ex.
./bootstrap-host.sh setup-all-in-one.yml nexus
```
