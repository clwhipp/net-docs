# Borg Backup

https://borgbackup.readthedocs.io/en/stable/

## Installation

```bash
sudo apt-get install borgbackup
```

## Setup

### Step 1 - Create the repository to house the archives

```bash
borg init --encryption=keyfile /mnt/backup/borg
```

### Step 2 - Create archives (backups) as desired

```bash
ARCHIVE_NAME='{hostname}-{now}'
INCLUDES='/mnt/critical/Secure_bitwarden /mnt/critical/nextcloud/data/brandi/files /mnt/critical/nextcloud/data/cameron/files'
borg create --compression zstd,10 --progress --list --verbose --stats /mnt/backup/borg::${ARCHIVE_NAME} ${INCLUDES}
```

### Step 3 - Backup Key

```bash
root@www:/mnt/backup# borg key export
usage: borg key export [-h] [--critical] [--error] [--warning] [--info] [--debug] [--debug-topic TOPIC] [-p] [--log-json] [--lock-wait SECONDS] [--bypass-lock] [--show-version]
                       [--show-rc] [--umask M] [--remote-path PATH] [--remote-ratelimit RATE] [--consider-part-files] [--debug-profile FILE] [--rsh RSH] [--paper] [--qr-html]
                       [REPOSITORY] [PATH]
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