---
title:  "Backup of Personal Photos"
tags: [rsync,bash,fstab,cron]
---
I have installed a Synology NAS server where my family store all photos on a **RAID 1+0** shared disk. Since the share is a raided system there is some redundancy but I want to have a backup on another computer for extra redundancy. So I have, at the moment, **rsync** mirroring the complete share to my lab computer as a cron job once every day.

First, let me show you how my share I mounted on my lab computer. Here is the relevant part of my **/etc/fstab**:

```bash
# /etc/fstab
/dev/sda1 /pictures-mirror     ext4    errors=remount-ro,ro 0       1
//192.168.88.246/photos /pictures cifs credentials=/home/nicklas/.smbcredentials,rw 0 0
```

The share is a **samba** share so that is why I specify my login credentials in a file (which is only readable by me or by root). The **/home/nicklas/.smbcredentials** could for example look like this:

```bash
username=my-username
password=my-secret-password
```

Here is the bash script that will do the **rsync**:


```bash
#!/usr/bin/env bash

display_help() {
  echo "Usage: $(basename "$0") SRC DEST " >&2
  exit 1
}

if [ "$#" -ne 2 ]; then
  display_help
fi

SOURCE=$1
DESTINATION=$2
LOG_FILE=/var/log/$(basename "$0").log

echo "Backing up pictures, date = $(date)" > ${LOG_FILE}

rsync --archive \
      --verbose \
      --delete \
      --human-readable \
      --progress \
      ${SOURCE} ${DESTINATION} >> ${LOG_FILE} 2>&1 

```

To run this daily I added the following to my **/etc/cron.d/backup-pictures**:

```
#<minute> <hour> <day_of_month> <month> <day_of_week> <user> <command>

06 22 * * * root /usr/local/bin/backup-pictures /pictures /pictures-mirror
```
