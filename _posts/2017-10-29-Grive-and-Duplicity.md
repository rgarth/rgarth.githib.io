---
layout: post
title: Using Google Drive as a backend for Duplicity
---

Dupicity can use Google Drive as a cloud backend for your Linux Workstation backups. The following works on Fedora.

### Install Required Software

```shell
$ sudo dnf install duplicity
$ sudo pip install pydrive
```

### Setup OAth Credentials in Google API
* Go to https://console.developers.google.com/
* Click "Create Project"
* Give it some sensible name like "Duplicity Backup"
* From the Dashboard click on "Enable API"
* Search for "Drive", select the Drive API, and enable it
* You will now need to create some credentials, Select "Credentials" from the panel on the left
* Use the arrow on the Create Credentials button to select "OAuth Client ID"
* You will need to give a Product Name. I used "Duplicity Backup"
* For Application Type, select "Other" and name the client. I used "Duplicity"
* Save your credentials somewhere safe (although they can easily be regenerated)

### Configure Duplicity
Setup some duplicity configs
```shell
$ cd ~
$ mkdir .duplicity
$ cd .duplicity
$ touch credentials
$ touch excludes
```

Put the following into your credentials file:
```
client_config_backend: settings
client_config:
   client_id: [id from previous step]
   client_secret: [secret from previous step]
save_credentials: True
save_credentials_backend: file
save_credentials_file: gdrive.cache
get_refresh_token: True
```

The excludes file will contain all the files you don't want backed up.
A good starting point:
```
**~
**.bak
**.cache
**cache**
**debuginfo**
**duplicity-**
**Trash**
**.iso
**/.cache/**
**/Cache/**
**/Downloads/**
**/BUILD/**
**/Music/**
**/temp/**
**/.thumbnails/**
**/.beagle/**
```

### Running Duplicity
---
_The first time you run duplicity it will return a URL, follow the link,
allow the application in your google account and paste in the code it returns._

_If you run from a different working directory you will be re-prompted to
authorize the app, so run from the same working directory each time._

---

Running the following command should now backup your home directory to Google
drive. The first run will be a full backup, every subsequent run will be incremental.

```shell
$ GOOGLE_DRIVE_SETTINGS=~/.duplicity/credentials duplicity --exclude-filelist ~/.duplicity/excludes ~/ gdocs://[username]@gmail.com/duplicity

```

### Extra Credit. A script to manage your backups
If you want a script to manage you backups. Create a new file "~/.duplicity/run-backup"
```shell
#!/bin/bash
##
# Shared configuration settings
#
#
# USE A GOOD LONG PASSPHRASE! DO NOT LOSE IT!
export PASSPHRASE="!!!set me!!!"
#
# 3 is a good verbosity level for daily runs, but you may want to start with
# 5 or above if you are just setting up
VERBOSITY=3
# If duplicity finds this file in any folder, that folder won't be backed up.
# You can specify separate LOCAL and OFFSITE settings below, but mind that you
# will need to touch both files in folders that you want to exclude from both.
EXCLUDE_IF_PRESENT=".nobak"
# Read the manpage for the exclude filelist format. You can specify
# different local and remote filelists in the sections below if you like.
EXCLUDE_FILELIST=$HOME/.duplicity/excludes
##
# OFFSITE backup configuration settings
#
# Set this to point to your Google Drive location
OFFSITE_DEST="gdocs://[username]@gmail.com/duplicity"
# You can specify a different offsite-specific exclude filelist here
OFFSITE_EXCLUDE_FILELIST=$EXCLUDE_FILELIST
# You can specify an offsite-specific exclude-if-present filename here
OFFSITE_EXCLUDE_IF_PRESENT=$EXCLUDE_IF_PRESENT
# You don't want to make full off-site backups too frequently, as this
# would require re-uploading full contents all over again. 90 days is a
# good metric
OFFSITE_FULL_EVERY="90D"
# You probably don't want to keep more than 1 full remote backup
OFFSITE_KEEP_FULL=1
# You should print out the contents of this file, in case you
# need to restore from off-site backups after a major disaster.
export GOOGLE_DRIVE_SETTINGS=$HOME/.duplicity/credentials
##
# LOCAL backup configuration settings
#
# Set this to point to your removable storage
# or comment out if you don't want local backups
LOCAL_DEST="/mnt/removable/duplicity"
# You can specify a different local-specific exclude filelist here
LOCAL_EXCLUDE_FILELIST=$EXCLUDE_FILELIST
# You can specify a local-specific exclude-if-present filename here
LOCAL_EXCLUDE_IF_PRESENT=$EXCLUDE_IF_PRESENT
# Create full backups every 30 days
LOCAL_FULL_EVERY="30D"
# Since you probably have lots of local room, keep 3 full backups
LOCAL_KEEP_FULL=3
##
# Perform an off-site backup
#
if [ ! -z "${OFFSITE_DEST}" ]; then
    # Clean up any previously failed runs
    duplicity --verbosity=0 cleanup --force ${OFFSITE_DEST}
    if [ $VERBOSITY -gt 0 ]; then
        echo "Performing an offsite backup to ${OFFSITE_DEST}"
    fi
    duplicity \
        --verbosity=$VERBOSITY \
        --allow-source-mismatch \
        --full-if-older-than=$OFFSITE_FULL_EVERY \
        --exclude-if-present=$OFFSITE_EXCLUDE_IF_PRESENT \
        --exclude-filelist=$OFFSITE_EXCLUDE_FILELIST \
        $HOME ${OFFSITE_DEST}
    if [ $VERBOSITY -gt 0 ]; then
        echo "Removing all but ${OFFSITE_KEEP_FULL} full backup from ${OFFSITE_DEST}"
    fi
    duplicity --verbosity=$VERBOSITY \
        remove-all-but-n-full ${OFFSITE_KEEP_FULL} \
        --force ${OFFSITE_DEST}
fi
##
# Perform a local backup
#
if [ ! -z "${LOCAL_DEST}" ]; then
    # If the dir does not exist, we'll assume that you forgot to mount the
    # external disk.
    if [ -d "${LOCAL_DEST}" ]; then
        # Clean up any previously failed runs
        duplicity --verbosity=0 cleanup --force "file://${LOCAL_DEST}"
        if [ $VERBOSITY -gt 0 ]; then
            echo "Performing a local backup to ${LOCAL_DEST}"
        fi
        duplicity \
            --verbosity=$VERBOSITY \
            --allow-source-mismatch \
            --full-if-older-than=$LOCAL_FULL_EVERY \
            --exclude-if-present=$LOCAL_EXCLUDE_IF_PRESENT \
            --exclude-filelist=$LOCAL_EXCLUDE_FILELIST \
            $HOME "file://${LOCAL_DEST}"
        if [ $VERBOSITY -gt 0 ]; then
            echo "Removing all but ${LOCAL_KEEP_FULL} full backup from ${LOCAL_DEST}"
        fi
        duplicity --verbosity=$VERBOSITY \
            remove-all-but-n-full ${LOCAL_KEEP_FULL} \
            --force "file://${LOCAL_DEST}"
    else
        echo "${LOCAL_DEST} not found. Did you forget to mount it?"
    fi
fi
```

