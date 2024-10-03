---
title: "Hosting a mirror"
date: 2023-07-24T18:14:30+02:00
draft: false
---

Mirrors are used by most Linux Distros to distribute packages and installer images.
This post documents the setup of an Arch Linux mirror.
The most common way to keep you mirror in sync is rsync

### Checking disk usage

```bash
rsync -avzhn rsync://mirror.lcarilla.de/archlinux/
```

Use this command to check this total size of an rsync module to check how much space you need on your server

### Web server setup

You need to set up an HTTP(S) server to distribute the files.
Make sure to enable directory listing to enable users to browse your mirror, for example when they're looking for installer images
If you're using the Apache Web server, put these two lines in your configuration

```
Options +Indexes
Options FollowSymLinks
```

### Set up a cronjob

A simple bash script that sync your mirror might look like this:

```
#!/bin/bash
#your sync home folder
SYNC_HOME="/var/www/vhosts/mirror.lcarilla.de"
#destination of the files you want to sync
SYNC_FILES="$SYNC_HOME/httpdocs/archlinux"
#rsync lock file - will be removed when finished
SYNC_LOCK="$SYNC_HOME/mirrorsync.lck"
#the server you want to sync from
SYNC_SERVER=rsync://ftp.halifax.rwth-aachen.de/archlinux/
MAX_BANDWIDTH=35000
rsync -rlptH \
        --delete-after \
        --bwlimit=$MAX_BANDWIDTH \
        --safe-links \
        --delay-updates $SYNC_SERVER "$SYNC_FILES"
sleep 5
rm -f "$SYNC_LOCK"
```

this crontab entry runs the script every 10 minutes

```
*/10 * * * * /var/www/vhosts/mirror.lcarilla.de/sync.sh
```
