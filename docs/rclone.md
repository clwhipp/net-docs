
# rclone
rclone is a utility that supports a wide variety of storage services for the movements of data. rclone is design
to work in a similar fashion to rsync and support many similar options. The rclone help menu can be opened with
the *--help* option.

Additional information on rclone can be found at https://rclone.org/.

```bash
root@www:/mnt/critical/Secure_maintenance/network-services# rclone --help

Rclone syncs files to and from cloud storage providers as well as
mounting them, listing them in lots of different ways.

See the home page (https://rclone.org/) for installation, usage,
documentation, changelog and configuration walkthroughs.

Usage:
  rclone [flags]
  rclone [command]

Available Commands:
  about           Get quota information from the remote.
  authorize       Remote authorization.
  backend         Run a backend-specific command.
  bisync          Perform bidirectional synchronization between two paths.
  cat             Concatenates any files and sends them to stdout.
  check           Checks the files in the source and destination match.
  checksum        Checks the files in the source against a SUM file.
  cleanup         Clean up the remote if possible.
  completion      Generate the autocompletion script for the specified shell
  config          Enter an interactive configuration session.
  copy            Copy files from source to dest, skipping identical files.
  copyto          Copy files from source to dest, skipping identical files.
  copyurl         Copy url content to dest.
  cryptcheck      Cryptcheck checks the integrity of a crypted remote.
  cryptdecode     Cryptdecode returns unencrypted file names.
  dedupe          Interactively find duplicate filenames and delete/rename them.
  delete          Remove the files in path.
  deletefile      Remove a single file from remote.
  genautocomplete Output completion script for a given shell.
  gendocs         Output markdown docs for rclone to the directory supplied.
  hashsum         Produces a hashsum file for all the objects in the path.
  help            Show help for rclone commands, flags and backends.
  link            Generate public link to file/folder.
  listremotes     List all the remotes in the config file.
  ls              List the objects in the path with size and path.
  lsd             List all directories/containers/buckets in the path.
  lsf             List directories and objects in remote:path formatted for parsing.
  lsjson          List directories and objects in the path in JSON format.
  lsl             List the objects in path with modification time, size and path.
  md5sum          Produces an md5sum file for all the objects in the path.
  mkdir           Make the path if it doesnt already exist.
  mount           Mount the remote as file system on a mountpoint.
  move            Move files from source to dest.
  moveto          Move file or directory from source to dest.
  ncdu            Explore a remote with a text based user interface.
  obscure         Obscure password for use in the rclone config file.
  purge           Remove the path and all of its contents.
  rc              Run a command against a running rclone.
  rcat            Copies standard input to file on remote.
  rcd             Run rclone listening to remote control commands only.
  rmdir           Remove the empty directory at path.
  rmdirs          Remove empty directories under the path.
  selfupdate      Update the rclone binary.
  serve           Serve a remote over a protocol.
  settier         Changes storage class/tier of objects in remote.
  sha1sum         Produces an sha1sum file for all the objects in the path.
  size            Prints the total size and number of objects in remote:path.
  sync            Make source and dest identical, modifying destination only.
  test            Run a test command
  touch           Create new file or change file modification time.
  tree            List the contents of the remote in a tree like fashion.
  version         Show the version number.

Use "rclone [command] --help" for more information about a command.
Use "rclone help flags" for to see the global flags.
Use "rclone help backends" for a list of supported services.
```

## Remote Management
Within rclone, data can be pushed or pulled from remotes that have been configured within the
system. For instance, pushing data into an S3 bucket would require that a dedicated remote be
created that points to Amazon AWS S3.

### List Remotes
The currently configured remotes within the system can be seen with the rclone listremotes command.

```bash
root@www:/mnt/critical/Secure_maintenance/network-services# rclone listremotes
iDriveE2NextcloudProd:
iDriveE2VaultwardenProd:
```

It's possible to determine where the file storing the configuration is located with the following command.

```bash
root@www:/mnt/critical/Secure_maintenance/network-services# rclone config file
Configuration file is stored at:
/root/.config/rclone/rclone.conf
```

### Creation
Adding remotes allows for data to be pulled or pushed to additional locations.

```bash
rclone config
```

The above command will open an interactive menu for managing the remotes within the system. For this section,
the 'n' option should be selected to add new targets.

```bash
root@www:/mnt/critical/Secure_maintenance/network-services# rclone config
Current remotes:

Name                 Type
====                 ====
iDriveE2NextcloudProd s3
iDriveE2VaultwardenProd s3

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
```

## Backup
Rclone can be utilized to push and pull data to the remotes that are configured. The pushing of data into the remote
can be utilized in a backup process. For instance, the results of a duplicity could be pushed into a remote S3 bucket
to support offsite backup operations.

The following is an example of pushing data from a duplicity backup to an iDrive E2 bucket. The appropriate fields will
need to be updated as required.

```bash
rclone copy --progress <source> <remote>:/<bucket name>
```

The credential programmed into the E2 remote must be provided with write access to the desired remote. The previous command
will pull the files from the source directory and upload them into the provided bucket. The following is an example related
to the backup of Vaultwarden to an E2 bucket called vaultwarden-duplicity-prod. The *--progress* flag tell rclone to provide
performance metrics as the upload process is in progress.

```bash
rclone copy --progress /mnt/backup/duplicity/vaultwarden iDriveE2VaultwardenProd:/vaultwarden-duplicity-prod
```

As with all backups, it's important to be able to acquire the files in the event a recovery becomes necessary. This is easily done
with rclone. The credentials programmed into the remote will need to have read access to the E2 bucket. The following is the pattern
for performing a restoration.

```bash
rclone copy --progress <remote>:/<bucket name>/<folder> <destination> 
```

In the previous example, the remote is the first argument in the copy operation. This tells rclone that the data should be downloaded
from the remote and stored wtihin the target destination.

```bash
rclone copy --progress iDriveE2VaultwardenProd:/vaultwarden-duplicity-prod/2022 /mnt/restore/vaultwarden/2022 
```
