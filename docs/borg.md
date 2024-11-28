# Borg Backup

Borg provides facilitites for backing up data through efficient
means through features such as deduplication and compression. Borg
places all backups for a given job within a repository. These
repositories then contain multiple archives (e.g. single backup).

More information can be found on the docs [here](https://borgbackup.readthedocs.io/en/stable/).

## Installation

Installation is easy and available by default within ubuntu repositories.

```bash
sudo apt-get install borgbackup
```

## Setting Up Backup

There are several steps to getting a borg repository configured and enabling
backups.

1. Creating Repository
2. Backing Keying Material
3. Create Archives

Each of the sections are discussed in greater detail.

### Creating Repository

A Borg repository contains the backup archives that are created. In general,
the repos will be encrypted with keys that are protected by a passphrase. There
are 2 options for creating the repository.

First, the wrapped key blob can be stored within the repo itself alongside
the archives. This simplifies the restoration process as the keys are within
the repository. This also means that the passphrase is only protection for data.

Repos with contained keys can be created with the following

```bash
borg init --encryption repokey /mnt/backup/borg
```

The second option is called *keyfile* as opposed to repokey. The keyfile solution will
install the wrapped keys into the server as opposed to the repo itself.
This results in more secure posture as the wrapped keys and the
corresponding passphrase would be required for unlocking the repos.

```bash
borg init --encryption keyfile /mnt/backup/borg
```

### Keying Material Backup

The keyfile option from previous step results in the keys being stored within the server.
As such, it also becomes necessary to backup the keys from the server so that it can be
restored should the server crash.

```bash
borg key export /mnt/backup/borg
```

Once exported, the key should be added to Bitwarden for archiving

### Create archives (backups) as desired

Borg calls a backup an archive within their documentation. Creating an archive
in Borg parlance is equivalent to running a backup. The following command
can be used to create a backup of */mnt/critical* into a repo located at
*/mnt/backup/borg*.

```bash
borg create                         \
    --verbose                       \
    --filter AME                    \
    --list                          \
    --progress                      \
    --stats                         \
    --show-rc                       \
    --compression zstd,10           \
    --exclude-caches                \
    --exclude /mnt/critical/ignore  \
                                    \
    /mnt/backup/borg::'critical-{now}' \
    /mnt/critical
```

## Recovery Process

**Steps**
1. Install borgbackup on client
2. Download borg repository from offsite location
3. Re-create keyfile within recovery system
4. Mount Recovery drive at same path (/mnt/backup/borg)

### Re-create keyfile from Bitwarden
The keyfile contains the decryption/MAC keys for the borg repository. The contents of the keyfile is encrypted
with a passphrase. The keyfile needs to be re-created on the system performing the recovery operation so that
it can decrypt the backup.

Open /root/.config/borg/keys/mnt_backup_borg in an editor, create if not present. The directory may need to be
created using the following command.

```bash
mkdir -p /root/.config/borg/keys
```

Once entered, save the contents of the file.

### Remount Borg Repository
The Borg repository needs to be mounted at same location for borg mapping to work correctly.

```bash
mkdir -p /mnt/backup
mount -t ext4 /dev/sdb1 /mnt/backup
```

**View Contents**
The *borg list* command provides a listing of the archives present in the backup.

```bash
root@thinkpad:borg list /mnt/backup/borg/
Enter passphrase for key /root/.config/borg/keys/mnt_backup_borg: 
vcenter-2022-12-01T21:07:04          Thu, 2022-12-01 21:07:04 [7b6e2497152015654c5b6aed44dbb81ed276b0218666969e74fc03fb2e81f0f4]
vcenter-2022-12-04T22:03:00          Sun, 2022-12-04 22:03:01 [dd441e60a28c58aa5e30fc2dec2345f00389b47626a378c33d25c53be52dbb74]
vcenter-2022-12-05T22:00:12          Mon, 2022-12-05 22:00:13 [8612f5b8a1f72176d492d1f7c78cd8db1a0b90bbb7eb41a796208e4f6a8dd432]
vcenter-2022-12-06T22:03:20          Tue, 2022-12-06 22:03:21 [a991b553e0ac8584e71297b4f284702a30c89d583f814362fe9fcbd89d97e5d8]
vcenter-2022-12-07T22:00:38          Wed, 2022-12-07 22:00:38 [49eddca0e4cd2b4c6c1f8096beb6a3f84e0220c6fdfc6a7a226eb1f3205d997d]
vcenter-2022-12-09T22:02:57          Fri, 2022-12-09 22:02:58 [5e5304f3f50a864729376b11d97b64d1d05b7846aa802c5de7b05ddfa8d7357a]
vcenter-2022-12-10T22:03:02          Sat, 2022-12-10 22:03:05 [8a37b7ed495dc42cc9cd7a6f867c3d03628a021c7b2807b34f877bbb0665bb7a]
vcenter-2022-12-11T22:02:17          Sun, 2022-12-11 22:02:18 [1ac856f2ba227912ac54b50b83a70129538c04a63e3c3ef9b63fbfb6a715d943]
vcenter-2022-12-12T22:00:16          Mon, 2022-12-12 22:00:17 [53a3cd34b5eaebe3716cd048b696125a077eba21ce239bf56be902d90f0b795b]
```
Each archive can be mounted into file-system and navigated.

```bash
mkdir -p /mnt/extract/borg
borg mount /mnt/backup/borg::<name from first column>
```