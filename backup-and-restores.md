# Backup and Restores
In case of factory reset, don't forget to backup this services. And then to restore them!

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

```