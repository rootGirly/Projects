# Restic: The Easy-Going Backup Tool

>Easy to set up, and fully automated. Two cups of tea later, you've got a powerful new tool in your toolbox.

Today, I felt like diving into **reliability, recoverability, and automation.**

Not because it sounds exciting on paper but because at some point, you realize that running a VPS without backups is basically hoping nothing goes wrong.

**And hope is not a strategy.**

![Restic logo](img/restic-logo.webp)
>I love the logo

Restic is a backup program that is fast, efficient and secure. It supports the three major operating systems (Linux, macOS, Windows) and a few smaller ones (FreeBSD, OpenBSD).

[Documentation](https://restic.net/)

## The big picture

My longtermn goal isn't just “make backups.” it's to build a system I could trust without thinking about it.

* I want backups to be created automatically
* I want them to be encrypted by default
* I want them stored offsite (not just on the same machine)
* I want old backups to be cleaned up intelligently (retention)
* And most importantly: I want to be able to restore everything if something breaks

I started locally, just to understand the process and make sure everything worked.

**But** : local backups are not enough for real reliability.

I would move toward an offsite setup using [SFTP](https://www.sftp.net/servers) in the comming days, because storing backups on the same server (or even same machine) is not a real safety net it only protects against small mistakes, not real failures.

### To keep things simple, I decided to build this setup with just a few tools:

* restic → my backup tool
* a local repository → only for testing and understanding the workflow
* cron jobs → to automate everything

## Installing Restic

For Ubuntu Machine : [Intallation](https://restic.readthedocs.io/en/stable/020_installation.html)

```
sudo apt update
sudo apt install restic
```

# Preparing a Repository (Local First, Migration Later)

The place where your backups will be saved is called a “repository”. This is simply a directory containing a set of subdirectories and files created by restic to store your backups, some corresponding metadata and encryption keys.

### Setting Up the Repository Location
For automation, restic allows you to define the repository location using an environment variable called:

 `RESTIC_REPOSITORY`

So I started by defining it:

`export RESTIC_REPOSITORY=/srv/restic-repo`

Then I created the directory:

```
mkdir -p /svr/restic-repo
chmod 700 /svr/restic-repo
```

**The permission is important:**

* it ensures only root can access the backups
* which is critical since they contain sensitive data

When initializing the repository, restic requires a password to encrypt all your backups.

Instead of choosing something manually, I generated a strong password directly from the terminal:

`openssl rand -hex 32`

Then I stored it in an environment variable:

```
export RESTIC_PASSWORD='some-strong-password'
```

**Note**: If your password contains special characters, the shell might interpret them in unexpected ways. To avoid that, it’s best practice to wrap the password in single quotes. 

[How strong is my password](https://www.security.org/how-secure-is-my-password/)


**Tips:** 
Verify what is currently set

```
echo "$RESTIC_PASSWORD"
echo "$RESTIC_REPOSITORY"
```

`unset RESTIC_PASSWORD` : to reset my password if needed.

### Deciding What to Back Up

What actually matters on my server?

* /etc → system config
* /var/www → websites/apps
* /home → user files


## Initializing My Backup Repository

```
restic init --repo /srv/restic-repo
enter password for new repository:
enter password again:
created restic repository 085b3c76b9 at /srv/restic-repo
Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.

```


## Creating my first backup

`sudo restic  --repo /srv/restic-repo backup /home/user`


```
repository ###### opened (version 2, compression level auto)
created new cache in /root/.cache/restic
no parent snapshot found, will read all files


Files:          33 new,     0 changed,     0 unmodified
Dirs:           12 new,     0 changed,     0 unmodified
Added to the repository: 1.182 MiB (270.663 KiB stored)

processed 33 files, 1.175 MiB in 0:00
snapshot ######## saved
```

Done! 


## Automate everything with my buddy cron job

A cron job is simply a scheduled task.

It tells my server: **Run this command every day at 2am**
Once it’s set up, it runs in the background automatically; no interaction needed.

**Why I Used It**

* I will forget to run backups manually
* automation = reliability


## Creating My Backup Script

`nano /root/scripts/backup.sh`

**Why use /root/backup.sh**

#### Access to everything

Backups need to read:

* /etc (system configs)
* /var/www (apps)
* /home (users)
* databases

script:

```
#!/bin/bash

# Run backup
restic -r /srv/restic-repo backup /etc /var/www /home /root/db-backup.sql

# Cleanup old backups
restic -r /srv/restic-repo forget --keep-daily 7 --keep-weekly 4 --keep-monthly 3 --prune
```

`chmod +x backup.sh`


## Scheduling It with Cron

Now the final step: telling the server when to run it.

`crontab -e`

And added:

`0 2 * * * /root/scripts/backup.sh`

Breaking It Down

```

0 2 * * *
│ │ │ │ │
│ │ │ │ └─ day of week
│ │ │ └── month
│ │ └──── day of month
│ └────── hour (2 AM)
└──────── minute (0)

```
this runs every day at 2:00 AM

Testing it

`bash /root/scripts/backup.sh`

Verifying the Backup

`restic -r /srv/restic-repo snapshots`

If everything is working correctly, restic will list the available snapshots with timestamps.

```
ID        Time                 Host        Tags        Paths
------------------------------------------------------------------
########  2026-04-14 18:39:43  ########                /home/user

########  2026-04-14 19:10:52  ########                /etc

########  2026-04-14 19:11:29  ########                /var/www

########  2026-04-14 19:37:05  ########                /etc
                                                       /home
                                                       /var/www
------------------------------------------------------------------
4 snapshots

```

Honestly, **restic is amazing.**

This whole setup was just for testing locally, but I didn’t expect it to be this smooth. I’m so used to struggling with tools...
But here, everything just worked.

**What’s Next**

Right now, this setup is local which is fine for learning, but not enough for **real reliability.**

So my next step is clear:

**set up an SFTP server for offsite backups**

Even though services like [BackBlaze](www.backblaze.com/) are cheap, I like the idea of building it myself first.

Not just to save money but to understand what’s actually happening behind the scenes.

**Be your own guru**