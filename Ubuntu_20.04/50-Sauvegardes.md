# Sauvegardes

Usage de [restic](https://restic.net/) pour sauvegardes chiffrées et snapshots

```
$ apt install restic
```

Usage de [autorestic](https://autorestic.vercel.app/) : un wrapper pour simplifier la configuration automatique de restic

```
$ wget -qO - https://raw.githubusercontent.com/CupCakeArmy/autorestic/master/install.sh | bash

$ vi  ~/.autorestic.yml

version: 2

global:
  forget:
    keep-daily: 10
    keep-weekly: 52
  options:
    backup:
      exclude:
        - '*.nope'
        - '*.swp'
        - '*.iso'

backends:
  hdd:
    type: local
    path: /backups
    key:
    options: {}

locations:
  etc:
    from:
    - /etc
    to:
    - hdd
    cron: '0 3 * * *' # Every day at 3:00
  home:
    from:
    - /home
    - /root
    to:
    - hdd
    cron: '0 3 * * *' # Every day at 3:00
    options: {}
  local:
    from:
    - /usr/local
    to:
    - hdd
    cron: '0 3 * * *' # Every day at 3:00
  www:
    from:
    - /var/www
    type: ""
    to:
    - hdd
    hooks:
      dir: ""
      before: []
      after: []
      success: []
      failure: []
    cron: 0 3 * * *
    options: {}
    copyoption: {}

$ autorestic check
...
Error: stat /var/www: no such file or directory
# /var/www sera créé plus tard
```

La clé (key) est générée si elle n'est pas définie : il faut la sauvegarder ``***SEC***``

```
$ crontab -e

...
PATH="/usr/local/bin:/usr/bin:/bin"
# every day at 03h00
0 3 * * * autorestic -c /root/.autorestic.yml --ci cron

# everty day at 02h20
20 2 * * * /usr/local/bin/backup.sh

```

## script backup.sh

sauvegarde de la liste des paquets

sauvegarde des bases de données en texte

```
$ vi /usr/local/bin/backup.sh

#!/bin/sh

/usr/bin/dpkg --get-selections > /home/sauv/dpkg-selec.txt

##############################################################################
# backup.sh
#
# .my.cnf contains
#
# [client]
# password   = my_passwd
##############################################################################

umask 066

# Sample for dumping database data 
#for database in `echo mysql information_schema somedb sympa`; do
#    /usr/bin/mysqldump --events --ignore-table=mysql.event --single-transaction -uroot $database > /home/sauv/mysqldump_$database.sql
#done;


$ chmod +x /usr/local/bin/backup.sh
```

## Usage autorestic

```
$ autorestic exec -av -- snapshots
$ autorestic exec -av -- diff 36e90d04 1c28bb1d

# restore file /root/.viminfo in tmp/
$ autorestic exec -av -- restore 1c28bb1d --target tmp/ --include /root/.viminfo
```

## Usage restic

il faut la clé

```
$ restic stats -r /backups/
$ restic snapshots -r /backups/
$ restic ls 855ec9cd -r /backups/
$ restic diff 36e90d04 1c28bb1d -r /backups/
```

## Ajout d'une clé de secours

```
$ restic key add -r /backups/

$ restic key list -r /backups/
enter password for repository: ***SEC***
repository f356d702 opened successfully, password is correct
 ID        User  Host   Created
-------------------------------------------
*2ddc3022  root  serv1  2022-04-19 15:24:44
 dca3ed76  root  serv1  2022-04-18 15:32:53
-------------------------------------------
```

# Sauvegarde externe sur disque/clé usb

Copie des sauvegardes chiffrées si une clé usb ou un disque usb avec un dossier racine ``[serv1]-backups/`` (``[serv1]`` est le hostname du serveur) est inséré.

### config udev

Pour conserver le montage si pas de dossier backup

```
$ vi /usr/lib/systemd/system/systemd-udevd.service
...
[Service]
MountFlags=shared
...
```

## règle udev

```
$ vi /etc/udev/rules.d/90-usb-backup.rules
ACTION=="add", SUBSYSTEM=="block", KERNEL=="sd[c-z]*", ENV{SYSTEMD_WANTS}="cpbackup@$env{DEVNAME}.service"

$ systemctl restart udev
```

``udev`` doit avoir un service ``systemd`` pour faire tourner un process plus long que le timeout par défaut de 2min 59s
```
$ vi /etc/systemd/system/cpbackup@.service
[Service]
Type=simple
TimeoutSec=0
GuessMainPID=false
ExecStart=/bin/bash -c "/usr/local/bin/usb-backup.sh %I"
```

## Script

1 beep au montage

2 beep à la fin de la copie

```
$ apt install beep
$ vi /usr/local/bin/usb-backup.sh

#!/bin/bash

VERBOSE="y"
MOUNT_POINT=/media/backup
DEVNAME=$1

log()
{
    if [ $1 != debug ] || expr "$VERBOSE" : "[yY]" > /dev/null; then
        logger -p user.$1 -t "usb-backup.sh[$$]" -- "$2"
    fi
}

log info "backuphd $DEVNAME"

if [[ ! -d "$MOUNT_POINT" ]]
then
   log info "mkdir -p $MOUNT_POINT"
   /bin/mkdir -p $MOUNT_POINT
fi

BACKUP_DIR="$(hostname)-backups"

modprobe pcspkr

if  [[ -b $DEVNAME ]] && mount $DEVNAME $MOUNT_POINT
then
   log info "Mount $DEVNAME"
   beep
   #echo "test" > $MOUNT_POINT/test.txt
   if [[ -d "$MOUNT_POINT/$BACKUP_DIR" ]]
   then
        echo $BACKUP_DIR
        log info "Copy /backups to $BACKUP_DIR"
        cp -pr /backups/* "$MOUNT_POINT/$BACKUP_DIR/"
        sleep 5
        log info "End of copy"
        /usr/bin/umount $DEVNAME
        beep
        beep
   fi
fi

$ chmod +x /usr/local/bin/usb-backup.sh
```

## logs

```
$ grep backup /var/log/syslog
```

## correction erreur de lecture sur disque sdc

```
$ fsck -p /dev/sdc1
```
