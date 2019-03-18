---
title: shell自动备份
date: 2019-03-18 17:58:48
tags:
---

原文
[https://www.admfactory.com/how-to-automatically-backup-files-and-directories-in-linux/](https://www.admfactory.com/how-to-automatically-backup-files-and-directories-in-linux/)

Backup script

## Step 1 – archive the content

Backing up your files using tar is very simple using the following command:

```
$ tar -cvpzf /backup/backupfilename.tar.gz /data/directory
```

A real example would be backing up the HTML folder for your website, my case:

```
$ tar -cvpzf /backup/admfactory.com.tar.gz /var/www/html
```

Note: make sure the destination folder exists first. If not create it using the following command:

```
$ mkdir /backup
```

Tar command explained
tar = Tape archive
c = Create
v = Verbose mode will print all files that are archived.
p = Preserving files and directory permissions.
z = This will tell tar that to compress the files.
f = It allows tar to get the file name.

## Step 2 – create backup script

Now let’s add tar command in a bash script to make this backup process automatic. Also it good to add some dynamic value in the name to make sure there is no overwriting of backup files. e.g.

Create the file using vi editor and paste below script.

```
$ vi /backup.sh
```

Paste the following script and change your details.

```
#!/bin/bash

TIME=`date +%b-%d-%y`                      # This Command will read the date.
FILENAME=backup-admfactory-$TIME.tar.gz    # The filename including the date.
SRCDIR=/var/www/html                       # Source backup folder.
DESDIR=/backup                             # Destination of backup file.
tar -cpzf $DESDIR/$FILENAME $SRCDIR
```

Note: I removed the v parameter for tar command line as is not needed.

Automation
In Linux, we can easily use the cron jobs in order to schedule task.

The cron jobs line has 6 parts see below explanation:

| Minutes |  Hours  | Day of Month |  Month  | Day of Week |    Command    |
| :-----: | :-----: | :----------: | :-----: | :---------: | :-----------: |
| 0 to 59 | 0 to 23 |   1 to 31    | 1 to 12 |   0 to 6    | Shell Command |

Open crontab editor utility:

```
$ crontab -e
```

Note: the edit rules are similar with vi editor.

Paste the following text in the editor:

```
# M H DOM M DOW CMND
00 04 * * * /bin/bash /backup.sh
```

This will run the script every day at 04:00:00.

For example, if you want to run the script only twice a week:

```
# M H DOM M DOW CMND
00 04 * * 1,5 /bin/bash /backup.sh
```

This will run the script at 04:00:00 every Monday and Friday.

Note: the only risk that can occur is to get out of disk memory if the source folder is big.
