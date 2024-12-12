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
