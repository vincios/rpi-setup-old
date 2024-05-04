# Backup and Restores
In case of factory reset, don't forget to backup this services. And then to restore them!

# Home
Don't forget to save user home directories

<details>
  <summary> Backrest configuration </summary>

  - Backup made each day at 01:00
  - Retention policy:
    - **Daily**: 7 - Keep a backup for the last 7 days
    - **Weekly**: 2 - Keep a backup for the last 2 weeks (the last one of each week)
      
      **NB**: one of them will coincide with the 7th daily backup (only the daily will be kept)

    - **Monthly**: 4 - Keep a backup for the last 4 months (the last one of each month)
      
      **NB**: one of them will coincide with the 7th daily or the 2nd weekly backup (only one will be kept)

    - **Yearly**: 1 - Keep a backup for the last year (the last one of the year)

  ```json
  {
    "id": "Home",
    "repo": "Synlogy-NetBackup",
    "paths": [
      "/home/raspi/Apps/instaloader",
      "/home/raspi/.backup",
      "/home/raspi/.bash_aliases",
      "/home/raspi/Bookshelf",
      "/home/raspi/Desktop",
      "/home/raspi/Documenti",
      "/home/raspi/Immagini",
      "/home/raspi/Modelli",
      "/home/raspi/Musica",
      "/home/raspi/Pubblici",
      "/home/raspi/Scaricati",
      "/home/raspi/Video",
      "/home/raspi/.config"
    ],
    "excludes": [],
    "iexcludes": [],
    "cron": "0 1 * * *",
    "backup_flags": [],
    "retention": {
      "policyTimeBucketed": {
        "yearly": 1,
        "monthly": 4,
        "weekly": 2,
        "daily": 7,
        "hourly": 0
      }
    }
  }
  ```
</details>

## Home Assistant 
On Home Assistant, create an automation to backup your configuration. For example, this automation will make a backup each Monday at 3AM

```yml
alias: ⚙️ [System] Backup each Monday
description: Make a configuration backup each Monday at 3AM
trigger:
  - platform: time
    at: "03:00:00"
condition:
  - condition: time
    weekday:
      - mon
action:
  - service: backup.create
    data: {}
mode: single
```

The compressed backups will be stored in the directory

```
/home/homeassistant/.homeassistant/backups
```

So, you only have to backup this folder.

To restore, Home Assistant core doesn't have a restore utility, so you have to manually extract the backup tar and replace the file inside the `.homeassistant` folder.


<details>
  <summary> Backrest configuration </summary>

  - Backup made each Monday at 03:30
  - Retention policy:
    - **Monthly**: 4 - Since the backup folder always contains all the oldest backups (except if manually removed), we just keep a snapshot for the last 4 months (the last one of each month)

  ```json
  {
    "id": "Home-Assistant",
    "repo": "Synlogy-NetBackup",
    "paths": [
      "/home/homeassistant/.homeassistant/backups"
    ],
    "excludes": [],
    "iexcludes": [],
    "cron": "30 3 * * 1",
    "retention": {
      "policyTimeBucketed": {
        "yearly": 0,
        "monthly": 4,
        "weekly": 0,
        "daily": 0,
        "hourly": 0
      }
    }
  }
  ```
</details>

## Immich
To backup and restore the Immich database, follow the official [wiki](https://immich.app/docs/administration/backup-and-restore).

If you have added the [postgres backup image](https://github.com/prodrigestivill/docker-postgres-backup-local) to your `docker-compose.yml` file (like stated in the wiki above), the dump will be automatically stored in the directory

```
your-immich-docker-folder/db_dumps
```

So, you only have to backup this folder (or better, the entire Immich docker folder).


## Backup utility
You can use this utility to schedule the backup of your Raspberry folders.

[RSYNC backup script](https://github.com/vincios/rsync_script)

Just clone the repository and follow the Readme to create the backup tasks.

### tasks.txt
This `tasks.txt` file is a good starting point to backup the services listed in this wiki.

⚠️ Make sure to adjust it with your paths!

```
# Backup Home Assistant
-> /home/homeassistant/.homeassistant/
@ /media/qnas/Vincenzo/Raspberry/Backups/homeassistant/
+backups/***
-*
<-

# Backup all the docker configurations
-> /home/raspi/dockers/
@ /media/qnas/Vincenzo/Raspberry/Backups/dockers/
<-

# Backup the rest of the raspi folder
-> /home/raspi/
@ /media/qnas/Vincenzo/Raspberry/Backups/raspi/
+Apps/
+Apps/instaloader/***
+.backup/***
+.bash_aliases
+Bookshelf/***
+Desktop/***
+Documenti/***
+Immagini/***
+Modelli/***
+Musica/***
+Pubblici/***
+Scaricati/***
+Video/***
-*
<-
```